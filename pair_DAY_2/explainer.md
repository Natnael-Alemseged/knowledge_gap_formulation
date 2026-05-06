# "Return JSON only" doesn't force JSON. Here's what actually forces it.

*Why instruction-following and constrained decoding are completely different mechanisms — and what it means for every pipeline that acts on structured LLM output.*

---

You have a judge LLM in your pipeline. You've told it:

> *"Return JSON only. No preamble, no explanation. Just the JSON object."*

It works great in testing. It works great in staging. Then in production it returns:

```
Sure! Here's my evaluation of the response:

{"score": 4, "reason": "The answer is mostly correct but..."}
```

Your `json.loads()` throws. Your pipeline catches nothing. Downstream code receives `None` and keeps running. Your evaluation scores are silently wrong for the next 200 requests before anyone notices.

Was this the model misbehaving? No. Was there ever a way to *actually* force JSON output? Yes — but it's not the prompt. Let me show you the real mechanism.

---

## What "return JSON only" actually does

When you write a format instruction in a prompt, you are doing exactly one thing: **shifting the probability distribution over the next token.**

The model has seen millions of examples during training where that kind of phrasing is followed by `{` and a well-formed JSON body. Your instruction loads that pattern strongly into the context. The probability mass on JSON-shaped tokens goes way up — often high enough that you get valid JSON 95–99% of the time on a well-tuned model.

But probable is not certain.

At every decoding step, the model selects the next token according to its output distribution. At temperature 0 it picks the argmax — the single highest-probability token — deterministically. At any temperature above 0 it samples, meaning lower-probability tokens can and do get selected. Either way, the instruction only shapes that distribution; it does not remove outcomes from it. A preamble phrase like `"Sure! Here's the evaluation:"` has a very small but non-zero probability at step one. If something in the context — a long system prompt, a conversational tone in your input, a model that was fine-tuned to sound helpful — nudges that probability even slightly upward, you get the preamble and your parse fails. Deterministic decoding reduces but does not eliminate the risk: if the highest-probability token at step one genuinely is a preamble token, you still get it.

This is instruction-following. It is a **soft mechanism**. It has no hard guarantees.

---

## What actually forces JSON: constrained decoding

There is a different mechanism called **constrained decoding** (also called structured generation or grammar-guided sampling). It does not operate at the prompt layer. It operates at the inference layer — before sampling happens.

Here is how it works:

At each decoding step, the system compares the current partial output against a grammar or schema. Any token that would make the output invalid at this parse state gets its logit set to **negative infinity** — probability zero. The model cannot produce that token. Not unlikely. *Cannot.*

The foundational paper is Willard & Louf (2023), [*Efficient Guided Generation for Large Language Models*](https://arxiv.org/abs/2307.09702). They show how to compile a JSON schema into a finite-state machine and use it to mask the vocabulary at each decoding step in O(1) time per token. That last part matters: the approach is fast enough to use in production without meaningful latency overhead.

This is implemented today in:

- **[Outlines](https://github.com/dottxt-ai/outlines)** — the reference library from the paper authors
- **llama.cpp** via `--grammar-file` (GBNF grammar format)
- **OpenAI structured outputs** (`response_format: { type: "json_schema", json_schema: {...} }`) — OpenAI's [documentation](https://platform.openai.com/docs/guides/structured-outputs) describes this as token-level schema enforcement, contracted to produce schema-valid output on every non-refused call. Note the qualifier: a safety refusal or content filter can still return a non-schema response — your boundary code should handle that case explicitly.

The difference from soft prompting is **categorical**, not quantitative. Instruction-following is a distribution shift. Constrained decoding is a hard exclusion.

---

## Soft vs hard: a minimal code comparison

**The soft approach — what most pipelines do:**

```python
import json

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": 'Evaluate this response. Return JSON only: {"score": int, "reason": str}'
    }]
)

try:
    result = json.loads(response.choices[0].message.content)
except json.JSONDecodeError:
    result = None  # silent failure — downstream receives None and keeps running
```

The `try/except` here is necessary but not sufficient. Catching the error and returning `None` just defers the damage — whatever uses `result` now has to handle `None` everywhere, and if it doesn't, the failure propagates silently and corrupts your scores.

**The hard approach — schema enforced at the token level:**

```python
from pydantic import BaseModel

class Evaluation(BaseModel):
    score: int
    reason: str

response = client.beta.chat.completions.parse(
    model="gpt-4o-2024-08-06",
    messages=[{"role": "user", "content": "Evaluate this response."}],
    response_format=Evaluation,
)

result = response.choices[0].message.parsed  # always a valid Evaluation — never None
```

No `try/except` on the parse. No `None` propagation. `result` is always a typed `Evaluation` object because the schema was enforced at the token level before the response was ever assembled.

---

## Back to my system: where this broke and what changed

In my LLM judge pipeline, the boundary parsing lives in `ledger/agents/credit_analysis_agent.py` (see the `_parse_json` helper). The utility function responsible for parsing judge output looked like this:

```python
def _safe_parse_json(raw: str) -> dict | None:
    try:
        return json.loads(raw)
    except json.JSONDecodeError:
        return None  # returned silently — the caller never knew
```

The failure case that exposed this: the judge model received an unusually long input passage and responded with a one-sentence acknowledgment before the JSON object. Here is a redacted example of the failing shape (synthetic but representative):

```
"Sure! Here's my evaluation:\n\n{\"score\": 0, \"reason\": \"...\"}"
```

`_safe_parse_json` returned `None`. The scoring loop treated `None` as a valid result, defaulted the score to `0`, and logged 47 evaluations as failures — all of them wrong, all of them silent.

The fix had two parts. First, the immediate boundary hardening:

```python
def _safe_parse_json(raw: str) -> dict:
    # Strip common preamble patterns before attempting parse
    start = raw.find("{")
    end = raw.rfind("}") + 1
    if start == -1 or end == 0:
        raise ValueError(f"No JSON object found in output: {repr(raw[:120])}")
    stripped = raw[start:end]
    try:
        return json.loads(stripped)
    except json.JSONDecodeError as e:
        raise ValueError(f"Judge returned unparseable output: {repr(raw[:120])}") from e
```

Second — and more importantly — the primary judge call was migrated to use `response_format` with a Pydantic schema. The stripping logic is now a fallback for open-weight model calls only. For the main judge endpoint, the parse cannot fail because the schema is enforced at decode time.

The model card was also updated to accurately reflect that the judge's output reliability comes from constrained decoding, not prompt engineering. That distinction matters the moment someone considers swapping the underlying model.

---

## One adjacent concept: this is how tool calling works too

The same token-level mechanism underlies provider-native function calling. When you send `tools=[...]` to the API, the model's tool-call output is not raw JSON that you hope parses — it is generated against a constrained schema enforced at the inference layer, exactly like `response_format`. This is why a trace showing `finish_reason: "tool_calls"` looks and behaves differently from a model generating JSON in the completion text: one is schema-enforced by the provider, the other is soft-prompted text your code then scrapes. A system that prompt-engineers JSON extraction and a system that uses real tool calling are not on a spectrum of "more or less agentic" — they are operating at categorically different layers of the stack.

---

## Three rules for any pipeline acting on structured LLM output

**1. Validate at every trust boundary.** Every point where LLM output enters your code as structured data is a trust boundary. Treat a parse failure as a first-class event — log it, alert on it, raise loudly — and never let a `None` flow silently downstream.

**2. Use constrained decoding when the output is load-bearing.** If a score, routing decision, or classification depends on structured output, use a constrained endpoint or library. Soft-prompt failures in the 1–5% range compound hard in multi-step pipelines. A judge that is wrong 2% of the time in isolation is wrong much more often when it runs 10 times in an evaluation chain.

**3. Keep the prompt instruction anyway.** Even with constrained decoding, write the format instruction in your prompt. It improves output quality and serves as documentation of intent for anyone reading the code. But treat it as a hint to the model, not a technical contract. The schema enforcement is the contract.

---

## The real lesson

The pipeline didn't break because the model was unreliable. It broke because the system was designed as if a prompt instruction were equivalent to a type constraint. It is not.

A prompt instruction is a statistical nudge. A grammar enforced at decode time is a guarantee. The moment structured LLM output feeds into code that acts on it — a scoring system, an agent router, a tool-call parser, an extraction pipeline — you need one of the two.

A nudge is not enough.

---

*The code in the "back to my system" section is drawn from a real LLM judge pipeline built during a structured AI engineering program. The failure described happened in production.*

---

**Sources**
- Willard, B. & Louf, R. (2023). *Efficient Guided Generation for Large Language Models.* arXiv:2307.09702. https://arxiv.org/abs/2307.09702
- OpenAI. *Structured Outputs — Platform Documentation.* https://platform.openai.com/docs/guides/structured-outputs