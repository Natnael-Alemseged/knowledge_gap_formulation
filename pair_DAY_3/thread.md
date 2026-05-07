# Tweet Thread: DPO vs SimPO in SalesConversion-Bench

**Tweet 1 / 6**

SalesConversion-Bench trains with TRL `DPOTrainer`.

Its methodology narrative argues SimPO-first.

That mismatch is not cosmetic: DPO and SimPO send different gradients through the same LoRA adapter on the same `(prompt, chosen, rejected)` pairs.

**Tweet 2 / 6**

DPO is reference-relative. It asks whether the policy increased the chosen-over-rejected log-prob gap beyond the reference model's gap:

`policy_gap - ref_gap`

Useful if the reference has good priors; risky if it already favors the wrong shortcut.

**Tweet 3 / 6**

SimPO is reference-free. It compares chosen vs rejected using average log-prob per token, then pushes that difference past target margin `gamma`.

That matters when answer lengths vary: total log-prob can make shorter answers look better for reasons unrelated to quality.

**Tweet 4 / 6**

The trap: a cleaner training curve does not mean a better judge.

Watch train margins against held-out behavior.

Bad pattern: `rewards/margins` rises on train, but held-out accuracy/margins stay flat or the model gets worse on buyer-safe qualitative slices.

**Tweet 5 / 6**

LoRA rank is a confounder, not the main story. Rank too low can make both objectives look weak. Rank too high can inflate train margins on tiny data.

Minimum clean matrix: DPO r=16, SimPO r=16, DPO r=8, SimPO r=8.

**Tweet 6 / 6**

Switch only on held-out evidence: 1+ extra pair, equal/better margins, no slice regression.

Otherwise: call this run DPO-LoRA; SimPO is the next ablation.

Full: https://dev.to/natnael_alemseged/dpo-vs-simpo-what-your-preference-trainer-is-actually-optimizing-42b4
