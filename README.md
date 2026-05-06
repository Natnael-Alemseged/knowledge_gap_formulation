# Week 12 — Knowledge Gap Formulation

**Topic:** Inference-time mechanics

---

## Day 1

**Pair:** Natnael Alemseged × Nebiyou Abebe

### My question

How does provider-side prompt caching actually work for my specific stack
(OpenRouter → qwen/qwen3-235b-a22b), and would my current prompt structure
qualify for a cache hit? See [`question.md`](pair_DAY_1/question.md).

### Explainer I wrote

**Why Merged LoRA Barely Changes Inference Time** — written for Nebiyou's
question about why a merged LoRA adapter produced nearly identical
inference latency to the base model.

### Public artifacts

| Artifact | URL |
|----------|-----|
| Blog post | https://dev.to/natnael_alemseged/why-merged-lora-barely-changes-inference-time-2mhj |
| Tweet thread | https://x.com/NotaZnation/status/2051669721011847592 |

### Daily deliverables

| File | Contents |
|------|----------|
| [`question.md`](pair_DAY_1/question.md) | Sharpened question with artifact pointer |
| [`morning_call_summary.md`](pair_DAY_1/morning_call_summary.md) | How both questions were sharpened in the morning call |
| [`explainer.md`](pair_DAY_1/explainer.md) | Blog post written for partner's LoRA latency question |
| [`thread.md`](pair_DAY_1/thread.md) | 6-tweet thread |
| [`sources.md`](pair_DAY_1/sources.md) | Canonical sources and benchmark tool used |
| [`evening_call_summary.md`](pair_DAY_1/evening_call_summary.md) | Feedback given and revisions made |
| [`signoff.md`](pair_DAY_1/signoff.md) | Gap-closure judgment on explainer received |
| [`grounding_commit.md`](pair_DAY_1/grounding_commit.md) | Pointer to CFO memo edit in Week 10 repo |

### Grounding commit

Edit to Week 10 CFO memo removing unverified caching claim:
https://github.com/Natnael-Alemseged/tenacious-conversion-engine/commit/8f687d5c6501bd5fb90991d5dc69c6eccf274436

---

## Day 2

**Pair:** Natnael Alemseged × Kirubel Gashaw

### My question

What is the exact mechanism by which an LLM emits a tool call when the API
request includes a `tools=[...]` schema — special token intercepted by the
provider, or JSON text parsed post-hoc? And how does this differ from my
current deterministic orchestration approach in reliability, latency, and
refusal behaviour? See [`question.md`](pair_DAY_2/question.md).

### Explainer I wrote

**"Return JSON only" doesn't force JSON. Here's what actually forces it.** —
written for Kirubel's question on the categorical distinction between
instruction-following (soft probability shift) and constrained decoding
(logit masking at inference time), grounded in a real silent-failure incident
in a production LLM judge pipeline.

### Public artifacts

| Artifact | URL |
|----------|-----|
| Blog post | https://dev.to/natnael_alemseged/return-json-only-doesnt-force-json-heres-what-actually-forces-it-9pn |
| Tweet thread | https://x.com/NotaZnation/status/2052107230044909959 |

### Daily deliverables

| File | Contents |
|------|----------|
| [`question.md`](pair_DAY_2/question.md) | Sharpened question with artifact pointer |
| [`morning_call_summary.md`](pair_DAY_2/morning_call_summary.md) | How both questions were sharpened in the morning call |
| [`explainer.md`](pair_DAY_2/explainer.md) | Blog post written for partner's structured output question |
| [`thread.md`](pair_DAY_2/thread.md) | 6-tweet thread |
| [`sources.md`](pair_DAY_2/sources.md) | Canonical sources used |
| [`evening_call_summary.md`](pair_DAY_2/evening_call_summary.md) | Session record and open items |
| [`signoff.md`](pair_DAY_2/signoff.md) | Gap-closure judgment and artifacts submitted |
| [`grounding_commit.md`](pair_DAY_2/grounding_commit.md) | Artifact edit record — `_safe_parse_json` before/after and judge migration |
