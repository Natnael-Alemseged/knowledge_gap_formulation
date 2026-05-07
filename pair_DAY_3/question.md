# Question — Week 12, Training and Post-Training Mechanics

## The Question

In `SalesConversion-Bench/memo.md:32`, the memo states that after SimPO tuning the LoRA judge still assigns the wrong log-probability ordering on dual-segment ICP preference pairs. Mechanically, how does SimPO compute the chosen-vs-rejected margin and propagate gradients through the LoRA adapter matrices — and given that mechanism, why would targeted contrast pairs fix this failure more directly than adding generic preference data at the same scale?

The third part is the load-bearing question. The first two are the context required to make the answer actionable.

The specific artifact is an ICP routing judge, but the underlying gap is general: how do SimPO gradients behave when the base model's prior strongly favors the rejected output? This is the exact situation any FDE encounters when fine-tuning a judge or critic on a small, skewed preference dataset — the project is the grounding, not the scope.

---

## Connection to Existing Artifact

**File:** `SalesConversion-Bench/memo.md`, paragraph 32

**Exact passage:**
> "After SimPO tuning, the LoRA adapter still assigns the wrong log-probability ordering on many dual-segment ICP preference pairs, so the trained judge prefers the weaker candidate when it must weigh two segment rules at once."

**Secondary files implicated:**
- `training/train_simpo.py:69` — uses `TRL CPOTrainer` with `loss_type="simpo"`. The default `cpo_alpha=1.0` is not overridden, so this run includes a chosen-output NLL regularizer. This is not clean standalone SimPO.
- `ablations/held_out_preference_summary.json:32` — source of the 33.3% ICP figure: `n=6`, `preference_accuracy=0.3333`, `mean_margin=-0.2647`. This is the held-out split, not train or dev.
- `ablations/held_out_preference_margins.jsonl:30` — the six ICP held-out rows. Four have negative margins; two succeed. All six are non-identical pairs.
- `tenacious_path_b_simpo_colab.ipynb:1578` — defines the failure criterion: `preference_margin = chosen_avg_logprob - candidate_avg_logprob`; failure is `margin <= 0`.
- `synthesis_memos/dpo.md:19` — claims SimPO's `gamma` replaces the KL regularizer, but does not explain what this means for pairs where the base model's prior strongly favors the rejected output.

---

## Why This Gap Matters to My FDE Work

The memo commits to a concrete deployment decision: block the trained critic for ICP routing until a targeted retraining run lifts the ICP slice from 33.3% to at least 80% accuracy. To defend that decision to a client or a senior engineer, I need to know whether the ICP failure is caused by:

- **Insufficient contrastive coverage** (the 91 pairs do not represent dual-segment ICP conflict at all), or
- **The SimPO margin objective** (the loss function cannot produce the right gradient direction for pairs where the base model's log-probability prior strongly favors the rejected output), or
- **LoRA adapter capacity** (rank 16 is not enough to shift the relevant projection matrices for this decision boundary), or
- **Beta/gamma settings** (the margin pressure is too loose to overcome a strong base-model prior), or
- **A problem the training approach cannot fix** (a deterministic routing rule would be more reliable than a trained judge for this slice regardless of data volume).

Each of these diagnoses implies a different next action. Without understanding the gradient mechanics, I cannot distinguish "add targeted ICP pairs" (principled fix) from "add targeted ICP pairs" (hopeful data patch that happens to work for other reasons). The deployment decision in the memo is defensible only if I know which diagnosis is correct.

---

## What a Satisfying Answer Looks Like

After reading the explainer I will be able to:

1. Read the SimPO training logs and explain, for a specific chosen/rejected pair, whether the gradient update moved the adapter in the right direction and why.
2. State the condition under which adding targeted ICP contrast pairs is a principled fix versus when a hyperparameter change or algorithm switch would be required instead.
3. Revise the memo's justification for the ICP retraining recommendation so it names the actual mechanism, not just the observed accuracy number.

An answer that explains SimPO loss math without connecting it to the ICP slice failure and the data-vs-hyperparameter decision will not close this gap.

---

## Context for the Explainer Writer

The following five questions were raised by my partner after reading the original draft. All five are answered here so the explainer can be written without guessing.

**1. Metric definition**
"Wrong ordering" means `preference_margin <= 0`, where:
```
preference_margin = chosen_avg_logprob - candidate_avg_logprob
```
The score is mean completion token log-probability — length-normalized by averaging over completion tokens after the prompt, not raw summed log-prob. All four ICP failures have negative margins; all six pairs are non-identical text. Source: `tenacious_path_b_simpo_colab.ipynb:1578`.

**2. Failure slice source**
The 33.3% figure comes from `ablations/held_out_preference_summary.json:32`. There are 6 held-out ICP pairs; 4 fail with negative margins, 2 succeed. This is the held-out split — not train or dev. The underlying rows are in `ablations/held_out_preference_margins.jsonl:30`. The held-out tasks live in `tenacious_bench_v0.2/held_out/tasks.jsonl`.

**3. Loss implementation**
This is not paper-faithful standalone SimPO. `training/train_simpo.py:69` uses `TRL CPOTrainer` with `loss_type="simpo"`. TRL's implementation computes the pairwise loss as approximately:
```
loss = -logsigmoid(beta * ((chosen_logp - rejected_logp) - gamma / beta))
```
But crucially: `cpo_alpha` defaults to `1.0` and is not overridden in `train_simpo.py`. This means a chosen-output NLL (behavior-cloning) regularizer is active. The run is more precisely: **TRL CPO with SimPO pairwise loss plus default NLL regularizer**, not clean SimPO. Pinned versions: `trl==1.3.0`, `transformers==5.6.2`, `peft==0.19.1`.

**4. Knobs in scope this week**
Per `ablations/icp_misclassification_refinement_playbook.md:38`, the scoped fix is **data only**: add 20–40 targeted ICP preference pairs, keep the original 91-pair file unchanged, rerun the same notebook with the augmented file.

Fixed for the clean refinement story: `beta=2.0`, `gamma=0.5`, `r=16`, `alpha=32`, same objective, same backbone, same notebook.

Technically changeable in code but outside the scoped Week 11 story: beta/gamma, LoRA rank, objective switch (DPO/ORPO), target modules, `cpo_alpha`.

Best short answer: **data first; objective and hyperparams held fixed**.

**5. Scope depth**
Confirmed: SimPO deep, DPO and ORPO as brief contrast only. No general algorithm survey needed.

---

## Scope Constraint for the Explainer

This question is not asking for a general comparison of DPO vs SimPO vs ORPO. It is asking about the gradient mechanics of SimPO specifically, in the context of a low-data preference fine-tune on a small LoRA, where the base model's prior conflicts with the desired output ordering. The adjacent concepts worth including are: how LoRA updates interact with the margin loss (not just LoRA in general), and the role of `beta` and `gamma` as levers on that specific interaction. Everything else is out of scope.

---

## Subtopic

Training and post-training mechanics — *The gradient mechanics of DPO vs SimPO vs ORPO; what each penalizes and rewards differently.*