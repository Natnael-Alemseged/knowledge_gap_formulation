# Day 4 — Morning Call Summary

**Participants:** Yohannes Dereje and Natnael Alemseged
**Duration:** ≥20 minutes

Natnael's original draft asked why P(mean_bootstrap ≤ 0) treats the bootstrap distribution of the estimator as the null
distribution of the test statistic — technically precise but too abstract to anchor in the artifact, so the call
sharpened it to name the 47 held-out tasks, the specific script file (`ablations/paired_bootstrap_delta_a.py`), the
broken line (`p_one_sided = sum(mean <= 0) / n_bootstrap`), and the concrete null hypothesis "the trained judge has no
true advantage over baseline on matched tasks." Yohannes's original question asked what mathematical property makes
pairing strictly necessary and what the unpaired CI would have been, but it was ambiguous on three axes: whether the
explainer should cover the variance-reduction mechanism or the experimental-design justification (or both), whether the
CI comparison should be theoretical or empirical, and what reviewer-facing conclusion was actually at stake. Natnael's
five clarifying questions resolved all three: Yohannes confirmed both the design justification and the
variance-reduction mechanism were needed (design justification first, as that is what a reviewer would challenge),
requested an empirical simulation using the raw vectors in `ablations/held_out_traces.jsonl`, and named the specific
claim under pressure — "training_wins: the LoRA lift is statistically significant above zero" — with the critical
boundary being whether the CI lower bound stays positive (a lower bound below zero changes the conclusion; a wider but
still-positive CI does not). Both questions were confirmed as passing the four-property rubric before research began.

---

**Confirmed by:** Natnael Alemseged

The above summary accurately reflects the call. My question was unambiguous to me by the end: the artifact pointer (
`ablations/paired_bootstrap_delta_a.py`), the specific line of broken code (
`p_one_sided = sum(mean <= 0) / n_bootstrap`), and the precise null hypothesis ("the trained judge has no true advantage
over baseline on matched tasks") were all named and locked before we separated to research.
