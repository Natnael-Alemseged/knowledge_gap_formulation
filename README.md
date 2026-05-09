# Week 12 — Knowledge Gap Formulation

**Topic:** Inference-time mechanics

---

## Review Index

This repo documents four completed paired research days and the portfolio edits they produced. Start with [`rules.md`](rules.md) for the submission rules and engineering decision criteria, [`synthesis.md`](synthesis.md) for the reflective summary, [`portfolio_update.md`](portfolio_update.md) for the Week 10/11 portfolio impact, and [`canonical_list.md`](canonical_list.md) for the papers, tools, and patterns used.

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

---

## Day 3

**Pair:** Natnael Alemseged × Hiwot Beyene

### My question

In `SalesConversion-Bench/memo.md:32`, the memo states that after SimPO
tuning the LoRA judge still assigns the wrong log-probability ordering on
dual-segment ICP preference pairs. Mechanically, how does SimPO compute the
chosen-vs-rejected margin and propagate gradients through the LoRA adapter
matrices — and given that mechanism, why would targeted contrast pairs fix
this failure more directly than adding generic preference data at the same
scale? See [`question.md`](pair_DAY_3/question.md).

### Explainer I wrote

**DPO vs SimPO: What Your Preference Trainer Is Actually Optimizing** —
written for Hiwot's question about resolving the mismatch between a
`DPOTrainer` code path and a SimPO-first methodology narrative in a small
LoRA preference-tuning run.

### Public artifacts

| Artifact | URL |
|----------|-----|
| Blog post | https://dev.to/natnael_alemseged/dpo-vs-simpo-what-your-preference-trainer-is-actually-optimizing-42b4 |
| Tweet thread | https://x.com/NotaZnation/status/2052491893473259629 |

### Daily deliverables

| File | Contents |
|------|----------|
| [`question.md`](pair_DAY_3/question.md) | Sharpened question with artifact pointer |
| [`question_peer.md`](pair_DAY_3/question_peer.md) | Hiwot's sharpened question answered by my explainer |
| [`morning_call_summary.md`](pair_DAY_3/morning_call_summary.md) | Morning sharpening notes |
| [`explainer.md`](pair_DAY_3/explainer.md) | Blog post written for Hiwot's DPO-vs-SimPO objective mismatch |
| [`thread.md`](pair_DAY_3/thread.md) | 6-tweet thread |
| [`sources.md`](pair_DAY_3/sources.md) | Canonical sources and log-inspection pattern used |
| [`evening_call_summary.md`](pair_DAY_3/evening_call_summary.md) | Evening feedback and revision record |
| [`signoff.md`](pair_DAY_3/signoff.md) | Gap-closure judgment and artifacts submitted |
| [`grounding_commit.md`](pair_DAY_3/grounding_commit.md) | Pointer to SalesConversion-Bench memo edit and what changed |

### Grounding commit

Edit to SalesConversion-Bench memo naming the unresolved ICP slice, wrong
log-probability ordering, targeted contrast-pair fix, and 80% re-check gate:
https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/565d98e

---

## Day 4

**Pair:** Natnael Alemseged × Yohannes Dereje

### My question

In my Week 11 benchmark, my script bootstraps 47 paired task outcomes and
reports `P(mean_bootstrap ≤ 0)` as a one-sided p-value. Why is that not a
valid hypothesis-test p-value, and what null distribution should I use instead
for the null hypothesis "the trained judge has no true advantage over baseline
on matched tasks"? See [`question.md`](pair_DAY_4/question.md).

### Explainer I wrote

**Why Pairing Your Bootstrap Is Necessary — And When It Stops Helping** —
written for Yohannes's question about what mathematical property of a
within-subject evaluation design makes paired bootstrap the correct procedure,
and whether an unpaired bootstrap would have changed his reviewer-facing
conclusion.

### Public artifacts

| Artifact | URL |
|----------|-----|
| Blog post | https://dev.to/natnael_alemseged/why-pairing-your-bootstrap-is-necessary-and-when-it-stops-helping-2iim |
| Tweet thread | https://x.com/NotaZnation/status/2052867037802713390 |

### Daily deliverables

| File | Contents |
|------|----------|
| [`question.md`](pair_DAY_4/question.md) | Sharpened question with artifact pointer |
| [`morning_call_summary.md`](pair_DAY_4/morning_call_summary.md) | How both questions were sharpened in the morning call |
| [`explainer.md`](pair_DAY_4/explainer.md) | Blog post written for partner's paired bootstrap question |
| [`thread.md`](pair_DAY_4/thread.md) | 6-tweet thread |
| [`sources.md`](pair_DAY_4/sources.md) | Canonical sources and tool used |
| [`evening_call_summary.md`](pair_DAY_4/evening_call_summary.md) | Feedback given and revisions made |
| [`signoff.md`](pair_DAY_4/signoff.md) | Gap-closure judgment on explainer received |
| [`grounding_commit.md`](pair_DAY_4/grounding_commit.md) | Pointer to McNemar fix across Week 11 eval scripts |

### Grounding commit

Replaced invalid bootstrap p-value with McNemar's exact test across
`paired_bootstrap_delta_a.py`, `run_ablations.py`, CFO memo, README,
evidence graph, and ablation summary:
https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/c841f47631bfa3df8d02e8bcc24a73b83adaac53
