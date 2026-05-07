# Evening call summary — Day 3 (training and post-training mechanics)

**Participants:** Hiwot Beyene, Natnael Alemseged
**Date:** 2026-05-07

---

By the time we met, we had already swapped final explainers and each landed a grounding commit in the real project (not just Week 12 notes). The evening call was deliberately about those artifacts: whether the words in the explainer match what git show proves, and whether a reviewer can trace claim → file → hash without trusting vibes.

We walked Hiwot's explainer against tenacious-bench: peers pressed on GitHub anchors (trainer path, for_unsloth vs preferences.jsonl, model_card percentages) and on the ablation matrix—is it minimal or still smuggling extra knobs? The grounding commit on docs/methodology_rationale.md + the Rafailov memo got the heat: does it fully retire "SimPO ran" as a reading, without pretending SimPO is discarded as a research direction? We agreed the honest line is DPO in the loop today, SimPO as a falsifiable next trainer, and the explainer's stop rule (held-out + margins/slices, not train margin alone) is what makes Day 3 decision-shaped, not just theory.

We did the same pass on Natnael's side: explainer vs his SalesConversion-Bench / trainer claims (TRL CPOTrainer + loss_type="simpo", default cpo_alpha NLL), and whether his grounding commit closes the margin / objective ambiguity his question named. The explainer correctly named both coverage starvation (more / better targeted ICP contrast) and objective deadlock (margin loss + strong prior → tune cpo_alpha / beta / gamma, or switch objective)—but did not commit to which story his logged pattern supports.

On the call Hiwot asked directly—this is the bar agreed to land in Natnael's next explainer revision:

> Given mean_margin = -0.2647 on four failures across six held-out ICP pairs, and eval / train accuracy at 1.0 on the training distribution while ICP held-out margins stay negative, which failure mode does that pattern most likely indicate: coverage starvation (a data fix is the principled first move), objective deadlock (a data-only patch is unlikely to work without objective / hyperparameter change), or adapter / margin-pressure limits (rank or beta/gamma ceiling)?

**Call reading of the pattern:** Train preference accuracy saturating at 1.0 while a small ICP held-out slice stays mostly wrong with negative margins is most consistent, as a first hypothesis, with coverage / representation gap: the run may be overfitting or shortcutting the training preference distribution without learning the dual-segment ICP decision boundary. Objective deadlock becomes the leading alternative if a targeted ICP augmentation (same trainer, honest eval) fails to move held-out ICP margins off ≤ 0. Until that experiment runs, leading with cpo_alpha / beta / gamma alone risks tuning around a missing-data problem.

**Revision lists:**

- Both sides: tighten one subsection, one link, one sentence that overclaimed.
- Natnael (outstanding): add one paragraph to the explainer that explicitly picks coverage starvation as the primary diagnosis for the ICP slice given the train-saturated vs ICP-failing split, and names the condition under which objective deadlock becomes the leading alternative.

**Outcome:** Both sides left with small revision lists. Shared bar: evening sign-off assumes the explainer + commit already exist—the call stress-tests portfolio defensibility, not first-draft scope.

---

**Confirmed by:** Natnael Alemseged
