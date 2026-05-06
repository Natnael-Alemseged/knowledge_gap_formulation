# Grounding Commit — Week 12

## Peer question answered

> "In my Week 11 pipeline, I told an LLM judge to 'return JSON only' and it usually did. But when it didn't, my pipeline broke silently. Does telling a model to return JSON actually *force* it to, or does it just make JSON more likely? What is the real mechanism, and what does it mean for any system that acts on structured LLM output?"

---

## Explainer published

**Title:** "Return JSON only" doesn't force JSON. Here's what actually forces it.
**URL:** https://dev.to/natnael_alemseged/return-json-only-doesnt-force-json-heres-what-actually-forces-it-9pn
**Published:** May 6, 2026

## Thread published

**Platform:** X (Twitter)
**URL:** https://x.com/NotaZnation/status/2052107230044909959

---

## Artifact edited

**File:** `ledger/agents/credit_analysis_agent.py`
**Function:** `_safe_parse_json`

### Before

```python
def _safe_parse_json(raw: str) -> dict | None:
    try:
        return json.loads(raw)
    except json.JSONDecodeError:
        return None  # returned silently — the caller never knew
```

**Failure behaviour:** when the judge model prepended a preamble to its JSON output, `json.loads()` threw, the function returned `None`, and the scoring loop silently defaulted every affected evaluation to a score of `0`. No error was raised. No log entry was written. The pipeline continued.

### After

```python
def _safe_parse_json(raw: str) -> dict:
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

**Change behaviour:** parse failures now raise loudly with the first 120 characters of the raw output attached. `None` can no longer propagate silently downstream. The function signature change (`dict | None` → `dict`) enforces this at the type level.

### Primary judge call

The primary judge call was additionally migrated from a plain `chat.completions.create()` with a prompt instruction to `client.beta.chat.completions.parse()` with a Pydantic `Evaluation` schema passed as `response_format`. Output validity is now enforced at the token level by OpenAI's structured outputs, not by the prompt. The `_safe_parse_json` function is retained as a fallback for open-weight model calls only.

---

## Model card update

The judge's model card was updated to reflect that output reliability is guaranteed by constrained decoding at the endpoint layer, not by the prompt instruction `"return JSON only"`. This distinction is documented so that any future model swap prompts a reassessment of whether the replacement endpoint supports schema-enforced structured output.

---

## Knowledge gap closed

| Gap | Resolution |
|---|---|
| "Does a prompt instruction force JSON?" | No. It shifts token probabilities. It is a soft mechanism with no hard guarantee. |
| "What actually forces JSON?" | Constrained decoding — logit masking against a grammar/schema at each decode step. Implemented in Outlines, llama.cpp grammar sampling, and OpenAI structured outputs. |
| "What does silent failure mean for a pipeline?" | Every boundary where LLM output enters code as structured data is a trust boundary. Parse failures must be first-class events — raised, logged, never silently swallowed. |

---

## Repository note

The artifact edits above live in the **Week 11 portfolio repository** (`ledger/agents/credit_analysis_agent.py`), not in this Week 12 documentation repository. The commit SHA should be linked here once the Week 11 repo branch is merged: `[add commit SHA]`. The before/after code shown is the exact diff — reviewers can verify by inspecting `git log --follow -p ledger/agents/credit_analysis_agent.py` in the Week 11 repo.

---

## Sources

- Willard, B. & Louf, R. (2023). *Efficient Guided Generation for Large Language Models.* EMNLP 2023; arXiv:2307.09702. https://arxiv.org/abs/2307.09702
- OpenAI. *Structured Outputs — Platform Documentation.* https://platform.openai.com/docs/guides/structured-outputs
