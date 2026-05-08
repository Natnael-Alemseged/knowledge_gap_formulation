# Thread: Why Pairing Your Bootstrap Matters — And When It Stops Helping

**Tweet 1**
A colleague asked why his bootstrap must resample the *same* task indices for both baseline and trained LoRA — and whether an unpaired bootstrap would change the reviewer's conclusion.

Pairing is correct by design, and in his data it barely moves the CI. Here's why.

---

**Tweet 2**
The key is experimental design.

The **same 48 tasks** were evaluated under both systems — each gives a paired outcome: `(baseline_score, trained_score)`.

Paired bootstrap: what if these 48 pairs were sampled from the task population?
Unpaired asks a different question entirely.

---

**Tweet 3**
The math makes the distinction precise.

Paired lift is `E[B − A]` — expected *within-task* lift.
SE = `sqrt((Var(A) + Var(B) − 2·Cov(A,B)) / n)`

Unpaired estimates `E[B] − E[A]` as if the two models ran on unrelated samples.
Same point estimate. Different uncertainty model.

---

**Tweet 4**
Pairing narrows the CI in proportion to `Cov(A, B)`.

Here: r(A, B) = 0.167 — near-zero.
The trained LoRA passes 26 tasks the baseline *fails*, pushing correlation toward zero.

Empirical CIs:
- Paired: [+35.4, +68.8] pp
- Unpaired: [+35.4, +66.7] pp

Practically identical.

---

**Tweet 5**
Reviewer conclusion: unaffected.

The claim is "the LoRA lift is significant above zero." Both CIs have a lower bound of +35.4 pp — far above zero. Neither method threatens the verdict.

Pairing is correct. It just doesn't help when the two systems fail on different tasks.

---

**Tweet 6**
When *does* pairing matter?

When hard tasks fail both models and easy ones pass both — high covariance — it can cut CI width by 30–50%.

Low correlation = large capability gap. Good news for the model, bad news for paired efficiency.

Full explainer ↓
https://dev.to/natnael_alemseged/why-pairing-your-bootstrap-is-necessary-and-when-it-stops-helping-2iim

---
