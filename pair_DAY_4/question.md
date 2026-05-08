# Day 4 Question

## Question

In my Week 11 benchmark, I evaluate two systems on the same 47 held-out tasks. For each task I record whether the
baseline succeeds and whether the trained judge succeeds, giving a paired binary outcome per task — e.g.
`(baseline=0, trained=1)`. My script bootstraps those 47 task-pairs, computes the mean paired lift in each resample, and
reports `P(mean_bootstrap ≤ 0)` as a one-sided p-value.

**Why is that not a valid hypothesis-test p-value, and what null distribution should I use instead if the null
hypothesis is "the trained judge has no true advantage over the baseline on matched tasks"?**

---

## Artifact connection

**File:** `ablations/paired_bootstrap_delta_a.py` (Week 11 repo)

The script computes:

```python
pairs = [(baseline_success, trained_success), ...]  # 47 entries
# ... resamples pairs with replacement ...
p_one_sided = sum(mean_bootstrap <= 0) / n_bootstrap
```

Knowing the correct null distribution would let me replace the current `p_one_sided` line with a statistically valid
test and update the benchmark write-up's significance claim accordingly.

---

## Why this gap is worth a colleague's day

| Property          | How this question satisfies it                                                                                                                        |
|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Diagnostic**    | The gap is precisely named: I am conflating the bootstrap percentile CI with a permutation-test null distribution.                                    |
| **Grounded**      | The misuse lives in a specific file and a specific line; the fix changes a concrete claim in my Week 11 portfolio.                                    |
| **Generalizable** | Paired-bootstrap significance is a recurring mistake in LLM eval papers; closing this gap helps anyone benchmarking two systems on the same task set. |
| **Resolvable**    | A 600–1,000 word explainer covering bootstrap CIs vs. permutation tests on paired binary data is tractable and sufficient to close the gap.           |
