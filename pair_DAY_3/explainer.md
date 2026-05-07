---
title: "DPO vs SimPO: What Your Preference Trainer Is Actually Optimizing"
published: true
description: "A practical way to tell whether a small LoRA preference-tuning run should stay on DPO or switch to SimPO."
tags: ai, llm, finetuning, machinelearning
---

SalesConversion-Bench had one uncomfortable preference-tuning mismatch: the code trained with TRL `DPOTrainer`, while the methodology narrative argued for SimPO.

That is not just a naming issue. DPO and SimPO turn the same `(prompt, chosen, rejected)` pair into different update signals. If the held-out lift is small, like 22.73% vs 18.18%, the project cannot honestly claim whether the model improved because DPO was the right objective, because LoRA rank constrained the update, or because training margins improved without robust held-out behavior.

The useful answer is not "DPO good, SimPO good, ORPO also good." The useful answer is:

> Compare the objectives under fixed conditions, control for LoRA rank, and keep the objective whose gains survive held-out evaluation instead of only improving training margins.

## The gradient difference

### DPO: reference-relative preference learning

DPO treats preference tuning as a comparison between two log-probability gaps:

```text
policy_gap = log pi_theta(chosen | prompt) - log pi_theta(rejected | prompt)
ref_gap    = log pi_ref(chosen | prompt)   - log pi_ref(rejected | prompt)

loss = -log sigmoid(beta * (policy_gap - ref_gap))
```

So the update asks:

> Has the trainable policy made the chosen answer more preferred than the rejected answer, beyond what the reference policy already believed?

That reference-relative part is the key. DPO does not only ask whether the chosen answer is more likely than the rejected answer. It asks whether the policy improved that preference gap relative to a reference model.

In a LoRA setup with TRL `DPOTrainer(ref_model=None)`, the exact reference handling depends on TRL and PEFT configuration. Some setups avoid loading a separate reference model and compute reference behavior by disabling adapters; others use a frozen reference copy. The implementation detail should be verified in the actual training stack.

But the conceptual point stays the same: **DPO is anchored to a reference policy**. That can be helpful if the base instruct model already has useful judgment priors. It can also preserve the wrong shortcut if the reference already favors short, generic, policy-shaped answers.

### SimPO: reference-free margin learning

SimPO removes the reference model and scores each answer using average log-probability per token:

```text
r(prompt, answer) = (1 / answer_length) * log pi_theta(answer | prompt)

loss = -log sigmoid(beta * (r_chosen - r_rejected - gamma))
```

The update asks:

> Has the policy made the chosen answer better than the rejected answer by at least the target margin `gamma`, using length-normalized scores?

That changes two things:

1. **No reference anchor:** SimPO directly pushes the chosen answer above the rejected answer. It does not ask whether the policy improved relative to the base model.
2. **Length normalization:** A long rejected answer is not punished merely because total log-probability accumulates over more tokens.

That second point matters in preference data where chosen and rejected answers differ in length. If the preferred answer is often shorter, a total-log-prob objective can make brevity look like quality. SimPO's average-log-prob reward reduces that artifact.

The falsifiable hypothesis is:

> If DPO's gains are mostly coming from reference-relative or length artifacts, then SimPO with the same data, seed, train steps, and LoRA rank should produce cleaner held-out margins and accuracy without increasing the train/eval gap.

### Where ORPO fits

ORPO combines a supervised chosen-answer term with an odds-ratio preference term. It should not be co-equal in this comparison. The live mismatch is DPO in code vs SimPO in the methodology.

ORPO becomes interesting if both DPO and SimPO are unstable, or if the model needs stronger behavior-cloning pressure toward chosen outputs. For this decision, it is a fallback, not the main branch.

## The overoptimization check

Training loss alone is not enough. In a small preference-tuning run, the warning pattern is:

```text
training preference margins improve,
but held-out accuracy or held-out margins do not improve.
```

The two metrics to inspect first are:

| Diagnostic | Why it matters | Bad sign |
|---|---|---|
| Train `rewards/margins` vs held-out pair accuracy or held-out margins | Separates real preference learning from training-set margin inflation | Train margins rise while held-out behavior stays flat or worsens |
| Chosen/rejected reward or log-prob movement | Shows whether improvement comes from lifting chosen answers, suppressing rejected answers, or drifting oddly from the reference | Rejected scores collapse while chosen quality does not improve |

If TRL logs `rewards/chosen`, `rewards/rejected`, and `rewards/margins`, use those directly. If it also logs policy/reference log-probs, inspect whether the DPO margin is improving because chosen answers are becoming more likely, or mainly because rejected answers are being pushed down.

The second case is not automatically reward hacking. It is a review flag. It needs held-out and qualitative confirmation.

## Hands-on pattern: inspect train vs eval margins

Before arguing that DPO or SimPO "worked," add a tiny log inspection step. The goal is not to prove overoptimization from one scalar. The goal is to force the comparison between training margins and held-out behavior.

```python
import json
from pathlib import Path


def load_jsonl(path):
    rows = []
    for line in Path(path).read_text().splitlines():
        line = line.strip()
        if line:
            rows.append(json.loads(line))
    return rows


def last_number(rows, *keys):
    for row in reversed(rows):
        for key in keys:
            value = row.get(key)
            if isinstance(value, (int, float)):
                return float(value)
    return None


def review_preference_run(train_log, eval_log=None):
    train = load_jsonl(train_log)
    midpoint = max(1, len(train) // 2)

    early_margin = last_number(
        train[:midpoint],
        "rewards/margins",
        "train_rewards/margins",
    )
    late_margin = last_number(
        train[midpoint:],
        "rewards/margins",
        "train_rewards/margins",
    )
    chosen = last_number(
        train[midpoint:],
        "rewards/chosen",
        "train_rewards/chosen",
    )
    rejected = last_number(
        train[midpoint:],
        "rewards/rejected",
        "train_rewards/rejected",
    )

    print(f"train margin: {early_margin} -> {late_margin}")
    print(f"late chosen/rejected rewards: {chosen} / {rejected}")

    if eval_log:
        eval_rows = load_jsonl(eval_log)
        eval_margin = last_number(
            eval_rows,
            "eval_rewards/margins",
            "rewards/margins",
        )
        eval_acc = last_number(eval_rows, "eval_accuracy", "accuracy")
        print(f"held-out margin: {eval_margin}")
        print(f"held-out accuracy: {eval_acc}")

    if early_margin is not None and late_margin is not None:
        if late_margin > early_margin and eval_log is None:
            print("Review flag: train margin improved. Confirm with held-out pairs.")
        elif late_margin <= early_margin:
            print("Review flag: weak training signal. Check rank, LR, or pair quality.")
```

The useful pattern is the comparison:

```text
train margins up + held-out behavior up    -> plausible improvement
train margins up + held-out behavior flat  -> likely training-set margin inflation
train margins flat                         -> weak signal, bad data, or too little capacity
```

## LoRA rank is a confounder

The current LoRA config is `r=16`, `alpha=32`, `dropout=0.05`. For a 0.5B model, that is plausible. The risk is not that `r=16` is obviously wrong. The risk is that rank can fake an objective conclusion.

| Rank failure | Expected pattern | First observable |
|---|---|---|
| Rank too low | Training margins plateau early, train loss barely moves, held-out accuracy is flat | `rewards/margins` and train loss |
| Rank too high for small data | Training margins keep improving while held-out accuracy or margins get noisy or worse | Train/eval margin gap |

So rank should stay in the ablation, but lightly. It is not the main theory. It is a control that prevents the false conclusion "SimPO lost" or "DPO won" when the real issue was adapter capacity.

## The smallest decisive ablation

The cleanest small matrix is 2 objectives x 2 ranks:

| Run | Objective | LoRA rank | Purpose |
|---|---:|---:|---|
| A | DPO | r=16 | Current baseline |
| B | SimPO | r=16 | Isolate objective change at current capacity |
| C | DPO | r=8 | Test whether lower rank regularizes DPO |
| D | SimPO | r=8 | Test whether lower rank regularizes SimPO |

Everything else should stay fixed: model, train/validation split, seed, pair data, max length, epochs or steps, learning rate, batch size, and evaluation script.

The decision rule:

```text
Prefer SimPO if:
SimPO beats DPO at the same rank on held-out accuracy or held-out margin,
and the gain is not paired with a larger train/eval margin gap.

Prefer DPO if:
DPO matches or beats SimPO at the same rank,
and DPO has a smaller train/eval gap or better qualitative behavior.

Prefer the lower rank if:
r=8 has slightly lower training margins but equal or better held-out behavior.
```

The important correction is sample size. With only 22 held-out pairs, a 3 percentage-point rule is too fine-grained because one example is about 4.5 percentage points.

A more defensible rule is:

> Switch objectives only if the winner improves by at least one additional held-out pair, has better or equal held-out margin behavior, and does not worsen the important qualitative failure slices.

One extra correct pair alone is a hint. One extra correct pair plus cleaner margins and no slice regression is a decision.

## Bottom line

The SalesConversion-Bench run should be described as **DPO preference tuning with LoRA**, not SimPO-first preference tuning.

The gap closes when the project can say:

1. DPO updates the model using a reference-relative chosen-vs-rejected margin.
2. SimPO updates the model using a reference-free, length-normalized target margin.
3. The objective choice should be decided by a controlled DPO-vs-SimPO ablation at fixed rank, with one lower-rank control to catch LoRA overfitting.

That turns "we prefer SimPO" from a narrative claim into an experiment the project can actually defend.

## Sources

- Rafailov et al. (2023), [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/abs/2305.18290)
- Meng et al. (2024), [SimPO: Simple Preference Optimization with a Reference-Free Reward](https://arxiv.org/abs/2405.14734)
- Hong et al. (2024), [ORPO: Monolithic Preference Optimization without Reference Model](https://arxiv.org/abs/2403.07691)
- Hugging Face TRL, [`DPOTrainer` documentation](https://huggingface.co/docs/trl/dpo_trainer)
