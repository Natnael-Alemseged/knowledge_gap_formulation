# Thread — X / Twitter

---

**Tweet 1**
"Return JSON only" in your prompt does NOT force JSON output.

It makes JSON more likely.

Those are completely different things — and the difference is why your LLM pipeline broke silently last week.

🧵

---

**Tweet 2**
“Return JSON only” just shifts token probabilities.

At temp=0 the model picks the argmax token at every step — deterministic, but not format-locked. At temp>0 it samples. Either way, if “Sure! Here's my eval:” has higher probability than `{` at step one, you get the preamble and `json.loads()` throws.

---

**Tweet 3**
The mechanism that actually forces JSON is called constrained decoding.

Before sampling each token, the system checks: would it violate the schema?

If yes → logit set to -∞ → probability zero → that token cannot be produced.

Not unlikely. Impossible.

---

**Tweet 4**
This is implemented in:
- Outlines (Willard & Louf 2023, arXiv:2307.09702)
- llama.cpp grammar sampling
- OpenAI structured outputs (response_format: json_schema)

Outlines compiles a JSON schema to an FSM and masks the vocab each step (O(1)/token).

---

**Tweet 5**
Rule: every time code consumes LLM output as *data* is a trust boundary.

Soft prompt → validate + fail loudly.
Load-bearing decision → use constrained decoding (not the prompt).

---

**Tweet 6**
The lesson isn’t “prompts are bad.”

A prompt is a statistical nudge. A decode-time schema is a contract.

Design like you know the difference.

Full explainer with code → https://dev.to/natnael_alemseged/return-json-only-doesnt-force-json-heres-what-actually-forces-it-9pn