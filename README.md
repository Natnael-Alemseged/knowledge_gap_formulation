# Week 12 — Knowledge Gap Formulation: Day 1

**Topic:** Inference-time mechanics
**Pair:** Natnael Alemseged × Nebiyou Abebe

---

## My question

How does provider-side prompt caching actually work for my specific stack
(OpenRouter → qwen/qwen3-235b-a22b), and would my current prompt structure
qualify for a cache hit? See [`question.md`](./question.md).

## Explainer I wrote

**Why Merged LoRA Barely Changes Inference Time** — written for Nebiyou's
question about why a merged LoRA adapter produced nearly identical
inference latency to the base model.

## Public artifacts

| Artifact | URL |
|----------|-----|
| Blog post | https://dev.to/natnael_alemseged/why-merged-lora-barely-changes-inference-time-2mhj |
| Tweet thread | https://x.com/NotaZnation/status/2051669721011847592 |

## Daily deliverables

| File | Contents |
|------|----------|
| [`question.md`](./question.md) | Sharpened question with artifact pointer |
| [`morning_call_summary.md`](./morning_call_summary.md) | How both questions were sharpened in the morning call |
| [`explainer.md`](./explainer.md) | Blog post written for partner's LoRA latency question |
| [`thread.md`](./thread.md) | 6-tweet thread |
| [`sources.md`](./sources.md) | Canonical sources and benchmark tool used |
| [`evening_call_summary.md`](./evening_call_summary.md) | Feedback given and revisions made |
| [`signoff.md`](./signoff.md) | Gap-closure judgment on explainer received |
| [`grounding_commit.md`](./grounding_commit.md) | Pointer to CFO memo edit in Week 10 repo |

## Grounding commit

Edit to Week 10 CFO memo removing unverified caching claim:
https://github.com/Natnael-Alemseged/tenacious-conversion-engine/commit/8f687d5c6501bd5fb90991d5dc69c6eccf274436
