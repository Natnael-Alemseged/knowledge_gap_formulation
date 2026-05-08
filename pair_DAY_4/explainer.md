# Why Pairing Your Bootstrap Is Necessary — And When It Stops Helping

*Explainer for Yohannes Dereje's Day 4 question*

---

Yohannes's `paired_bootstrap` function in `ablations/run_ablations.py` (lines 271–310) resamples one set of 48 task indices and applies it to both the trained LoRA scores and the baseline scores. He asked two connected things: what mathematical property makes that correct, and whether an unpaired bootstrap would have changed the reviewer-facing conclusion.

The short answer: pairing is correct *by experimental design*. When the two score vectors have positive covariance, pairing usually reduces the model-based standard error; in Yohannes's specific data the correlation is near-zero (r = 0.167), so the paired and unpaired bootstrap CIs are practically identical — and neither changes his reviewer-facing conclusion.

Here is why, from first principles.

---

## The experimental design justification: why pairing is valid at all

The 48 held-out tasks were not drawn independently for the baseline and then re-drawn independently for the trained LoRA. The **same 48 tasks** were evaluated under all three variants. Each task is a repeated measurement on the same subject — this is the within-subject structure, and it is what makes pairing the correct procedure.

If the 48 baseline tasks and the 48 trained-LoRA tasks were *different* tasks drawn from the same population, unpaired bootstrap would be correct. But here, resampling index 13 means "draw task 13 for both models together." Resampling each vector independently breaks that structure and estimates uncertainty for a different experiment: baseline and LoRA evaluated on unrelated task samples.

This distinction matters before any formula. The formula follows the design; the design is what you defend to the reviewer.

---

## The variance-reduction mechanism: the math behind why pairing helps

Once you have established that pairing is correct, the question is how much it helps. The standard error of the mean paired lift is:

```
SE_paired = sqrt((Var(A) + Var(B) − 2·Cov(A, B)) / n)
```

where A is the baseline binary score vector, B is the trained-LoRA binary score vector, and n = 48.

The unpaired standard error treats A and B as independent, so the covariance term drops:

```
SE_unpaired = sqrt((Var(A) + Var(B)) / n)
```

The mathematical object is the joint distribution of paired outcomes, not two independent marginal pass rates. A paired design estimates `E[B - A]`: expected within-task lift. An unpaired design estimates `E[B] - E[A]` as if the two means came from unrelated samples. Same point estimate, different uncertainty model.

Pairing helps in proportion to the covariance between the two score vectors. If tasks where the baseline passes tend also to be tasks where the trained model passes, the covariance is large and positive, the numerator under the square root shrinks, and the paired SE is meaningfully smaller. If the two models fail and pass on largely *different* tasks — low covariance — pairing buys you almost nothing in precision, even though it remains the correct design.

### Yohannes's actual numbers

One caveat before the numbers: `ablation_results.json` reports a +26.4 pp lift with CI [+18.7, +32.8], while `held_out_traces.jsonl` gives +52.1 pp with CI [+35.4, +68.8]. This explainer uses `held_out_traces.jsonl` because it contains raw task-level vectors. Before this becomes reviewer-facing text, Yohannes should reconcile which file reflects the real Colab run.

From `ablations/held_out_traces.jsonl`:

| | Trained LoRA passes | Trained LoRA fails |
|---|---------------------|---------------------|
| **Baseline passes** | 15 | 1 |
| **Baseline fails** | 26 | 6 |

- Baseline: 16 passes, 32 fails across 48 tasks → proportion p_A = 16/48 = **0.333**
- Trained LoRA: 41 passes, 7 fails → proportion p_B = 41/48 = **0.854**
- Pearson r(A, B) = **0.167**
- Var(A) = 0.333 · 0.667 = **0.222** ; Var(B) = 0.854 · 0.146 = **0.125** ; Va + Vb = **0.347**
- Cov(A, B) = r · sqrt(Va · Vb) = 0.167 · sqrt(0.222 · 0.125) ≈ **0.028**

The task-level difference vector makes the paired structure visible:

- `+1` on 26 tasks where trained LoRA passes and baseline fails
- `-1` on 1 task where baseline passes and trained LoRA fails
- `0` on 21 tasks where both systems agree

The paired bootstrap resamples this population of task-level differences. The unpaired bootstrap destroys these `+1`, `-1`, and `0` relationships by drawing baseline and trained outcomes independently.

Plugging in:

```
SE_paired   = sqrt((0.347 − 2·0.028) / 48) = sqrt(0.291 / 48) ≈ 0.0779
SE_unpaired = sqrt(0.347 / 48)             ≈ 0.0850
```

The paired SE is about **8.4% smaller** than the unpaired SE — real but modest, because the covariance is small relative to `Var(A) + Var(B)`.

### Empirical simulation

Running both bootstrap methods on the exact score vectors:

```python
import numpy as np

rng = np.random.default_rng(42)
n_boot = 100_000

baseline = np.array([1]*15 + [1] + [0]*26 + [0]*6)
trained  = np.array([1]*15 + [0] + [1]*26 + [0]*6)

paired = []
unpaired = []
for _ in range(n_boot):
    idx = rng.integers(0, 48, 48)
    i_a = rng.integers(0, 48, 48)
    i_b = rng.integers(0, 48, 48)
    paired.append((trained[idx] - baseline[idx]).mean())
    unpaired.append(trained[i_b].mean() - baseline[i_a].mean())

print(np.percentile(paired, [2.5, 97.5]) * 100)
print(np.percentile(unpaired, [2.5, 97.5]) * 100)
```

Empirical results from `held_out_traces.jsonl`:

```
Paired CI:   [+35.4, +68.8] pp  — width 33.3 pp
Unpaired CI: [+35.4, +66.7] pp  — width 31.3 pp
```

The two CIs are essentially identical, which is what the near-zero covariance predicts. The SE formula says pairing should modestly reduce the model-based standard error, but percentile bootstrap CIs on binary-difference distributions are not symmetric ±1.96·SE intervals. Their tails can shift independently because the empirical distribution is discrete and skewed. So the slight width inversion is not a contradiction: pairing is still the right design, but here it does not buy meaningful precision.

---

## Does the difference change the reviewer conclusion?

Yohannes's reviewer-facing claim is:

> *"The LoRA adapter's lift is statistically significant above zero."*

The critical boundary is whether the CI lower bound stays positive. Both paired and unpaired bootstrap give a lower bound of **+35.4 pp** — far above zero. Neither variant threatens the `training_wins` verdict. A CI of [−2, +54] would change the conclusion; [+12, +40] would not (it would only reduce precision on the magnitude estimate). The actual data stays nowhere near the dangerous boundary regardless of which bootstrap method is used.

**The practical answer: pairing is correct by experimental design, and in this specific experiment it makes no difference to the reviewer conclusion — because the near-zero correlation between the two score vectors means pairing provides almost no variance reduction.**

---

## Adjacent concepts worth connecting

**When does pairing matter most?** When tasks are heterogeneous in difficulty and both models are sensitive to that difficulty. If hard tasks fail both models and easy tasks pass both, r(A,B) is large, the covariance term is large, and paired bootstrapping can cut CI width sharply. In Yohannes's data, the dominant pattern is trained LoRA passing where baseline fails, which pushes correlation toward zero and makes pairing nearly irrelevant for variance reduction.

**When does low correlation arise in LLM evals?** Near-zero r(A,B) can signal a large capability gap between systems: the stronger model succeeds on tasks that are too hard for the weaker one, so their pass/fail patterns decorrelate. That is good news for the trained model's lift, but it means paired bootstrapping loses most of its statistical efficiency advantage.

---

## Pointers

**Papers and primary sources:**
- Efron & Hastie, *Computer Age Statistical Inference* (2016), Ch. 11 — authoritative treatment of bootstrap CIs for paired designs; freely available via Stanford.
- Dror et al., "Replicability Analysis for Natural Language Processing: Testing Significance with Multiple Datasets" (TACL 2017) — canonical reference for paired bootstrap and permutation tests in NLP evaluation, directly applicable to LLM benchmarks.

**Tool used:**
- NumPy `default_rng` + bootstrap loop on `held_out_traces.jsonl` — reproducible in a Colab cell with no additional dependencies.

**Follow-on directions:**
- For a valid one-sided p-value (not just a CI), use a paired permutation test: randomly flip the sign of each task-pair's difference, compute the mean under the null, and count how often the null mean exceeds the observed mean. The bootstrap percentile CI lower bound being positive is consistent with significance but is not a p-value.
- The pairing argument holds under either data source, but the reviewer-facing claim must cite one consistent run.

