# Grounding Commit — Day 3

## Named gap

In SalesConversion-Bench, the memo reported that the SimPO-tuned LoRA judge still assigned the wrong log-probability ordering on dual-segment ICP preference pairs. The gap was not just that the model failed; it was understanding whether the failure called for targeted data, objective tuning, or a different preference algorithm.

---

## Public artifacts

**Explainer:** DPO vs SimPO: What Your Preference Trainer Is Actually Optimizing  
**URL:** https://dev.to/natnael_alemseged/dpo-vs-simpo-what-your-preference-trainer-is-actually-optimizing-42b4

**Thread:** https://x.com/NotaZnation/status/2052491893473259629

---

## Artifact edited

**Repository:** `SalesConversion-Bench`  
**Remote:** `git@github.com:Natnael-Alemseged/SalesConversion-Bench.git`  
**Commit:** `565d98e`  
**URL:** https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/565d98e  
**File:** `memo.md`  
**Line:** `memo.md:32`

The edited memo names the unresolved ICP failure precisely: after SimPO tuning, the LoRA adapter still gives the wrong log-probability ordering on many dual-segment ICP preference pairs. It records the held-out slice result as 2/6 correct, or 33.3%, with mean margin -0.26.

---

## What changed and why

The memo no longer treats the trained critic as globally deployable. It separates the categories where the critic reached 100% held-out performance from the unresolved ICP routing slice, then gates ICP deployment on a targeted retraining run.

The important wording is that the next step is not "more data" in general. It is targeted contrast pairs that isolate segment-rule tradeoffs, followed by the same held-out slice re-check. The acceptance gate is explicit: ICP must reach at least 80% held-out preference accuracy without regressing the categories that are already at 100%.

This edit is the portfolio payoff from the Day 3 gap. The explainer clarified why training objective, LoRA rank, and margin behavior must be separated before claiming that SimPO itself failed. The memo now states a falsifiable next move: try targeted ICP coverage first; if the slice still has negative margins after targeted augmentation, then objective deadlock or hyperparameter pressure becomes the leading diagnosis.

---

## Verification command

```bash
git -C /Users/natnaelalemseged/code-projects/backend/SalesConversion-Bench blame -L 32,32 -- memo.md
```

Expected anchor:

```text
565d98e1 ... The one unresolved training failure is **ICP misclassification** ...
```
