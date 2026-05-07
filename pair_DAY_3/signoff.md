# Week 12 Sign-off — Day 3

**Date:** May 7, 2026  
**Partner:** Hiwot Beyene  
**Topic:** Training and post-training mechanics

---

## Peer exchange

**Question Natnael answered (Hiwot's):** In Hiwot's `tenacious-bench` setup, the code path uses TRL `DPOTrainer` with LoRA while the methodology narrative argues SimPO-first. What gradient-level difference matters between DPO and SimPO, what overoptimization signals should be trusted, how does LoRA rank confound the interpretation, and what minimum ablation would make the DPO-vs-SimPO choice evidence-based?

**Status:** Closed. The final explainer gives the gradient-level distinction, treats ORPO as a fallback rather than a co-equal tangent, adds a hands-on log-inspection pattern, and proposes a controlled DPO/SimPO x rank matrix with a held-out-aware decision rule.

**Question Hiwot answered (Natnael's):** In SalesConversion-Bench, why does the SimPO-tuned LoRA judge still assign the wrong log-probability ordering on dual-segment ICP preference pairs, and when are targeted ICP contrast pairs the principled fix versus an objective or hyperparameter change?

**Status:** Closed for the Day 3 decision. The evening call concluded that the current logged pattern most strongly supports coverage starvation as the first hypothesis, with objective deadlock becoming the leading alternative only if targeted ICP augmentation fails to move held-out margins.

---

## Artifacts submitted

| File | Description |
|---|---|
| `question.md` | Natnael's sharpened SalesConversion-Bench gap |
| `question_peer.md` | Hiwot's sharpened DPO-vs-SimPO gap answered by Natnael |
| `explainer.md` | Published dev.to explainer answering Hiwot's question |
| `thread.md` | Published 6-tweet X thread |
| `sources.md` | Canonical papers, TRL documentation, and hands-on log-inspection pattern |
| `evening_call_summary.md` | Call summary, critique, and final decision framing |
| `grounding_commit.md` | Pointer to the SalesConversion-Bench memo edit tied to Natnael's named gap |

---

## Public links

- Blog post: https://dev.to/natnael_alemseged/dpo-vs-simpo-what-your-preference-trainer-is-actually-optimizing-42b4
- X thread: https://x.com/NotaZnation/status/2052491893473259629

---

## Knowledge gaps closed

**Objective-mechanics gap:** DPO is reference-relative; SimPO is reference-free and length-normalized. The two objectives can produce different update signals on the same preference tuples, so the trainer actually used must be named precisely.

**Evaluation gap:** Training margin improvements are not enough. The decisive signal is whether held-out accuracy, held-out margins, and qualitative failure slices move with the training margin rather than diverging from it.

**Ablation-design gap:** LoRA rank can make an objective look better or worse by changing adapter capacity. The minimum defensible comparison is DPO r=16, SimPO r=16, DPO r=8, and SimPO r=8 under fixed data, seed, steps, and evaluation.

**SalesConversion-Bench grounding gap:** The ICP slice failure is best treated first as a coverage problem: add targeted dual-segment ICP contrast pairs and rerun the same evaluation. Objective or hyperparameter changes become the leading fix only if that targeted data does not move held-out margins off the failure boundary.

---

## Sign-off

**Gap-closure judgment:** Closed.

The Day 3 explainer and thread are published, grounded in canonical sources, include a concrete hands-on inspection pattern, and connect back to a real SalesConversion-Bench artifact edit. The conceptual gap is closed: the remaining work is the next experimental run, not more explanatory clarification.

**Natnael Alemseged**  
Week 12 — Training and post-training mechanics  
May 7, 2026
