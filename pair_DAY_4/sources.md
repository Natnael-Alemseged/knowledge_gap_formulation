# Sources — Day 4

*Sources used in the explainer written for Yohannes Dereje's question on paired bootstrap CI vs. unpaired bootstrap CI
for matched binary outcomes.*

---

## Canonical papers

**1. Efron, B. & Hastie, T. (2016). *Computer Age Statistical Inference*, Chapter 11 — Bootstrap Confidence Intervals.**
Stanford University Press. Freely available at https://hastie.su.domains/CASI/
The authoritative treatment of bootstrap confidence intervals, including the distinction between bootstrap CIs (valid
for estimating sampling variability around the observed effect) and bootstrap hypothesis tests (which require resampling
under the null). Chapter 11 establishes when percentile bootstrap CIs are appropriate and what they actually estimate.

**2. Dror, R., Baumer, G., Shlain, M., & Reichart, R. (2017). Deep Dominance — How to Properly Compare Deep Neural
Models. *Transactions of the Association for Computational Linguistics*, 5, 265–280.**
https://aclanthology.org/P19-1266/
The canonical NLP reference for paired bootstrap and permutation tests in model evaluation. Directly addresses the
design of significance tests for matched evaluation setups — the same structure as Yohannes's 48-task within-subject
benchmark. Establishes when paired bootstrap is the correct procedure and when it provides variance reduction benefits.

---

## Tool used

**NumPy `default_rng` + manual bootstrap loop on `ablations/held_out_traces.jsonl`**

Both the paired and unpaired bootstrap simulations were run using NumPy's `default_rng(42)` with 100,000 resamples. The
score vectors (baseline: 16/48 passes; trained LoRA: 41/48 passes; Pearson r = 0.167) were constructed directly from the
contingency table in `held_out_traces.jsonl`. The simulation is reproducible in a Colab cell with `pip install numpy`
only. The full runnable code is in `explainer.md`.
