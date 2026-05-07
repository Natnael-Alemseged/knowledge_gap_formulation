# Sources — DPO vs SimPO vs ORPO: Gradient Mechanics

## Primary papers (two canonical sources)

**Rafailov et al. (2023).** *Direct Preference Optimization: Your Language Model is Secretly a Reward Model.* NeurIPS 2023.
- https://arxiv.org/abs/2305.18290
- **Why cited:** DPO gradient derivation and the implicit reward model equivalence (Equation 7). Load-bearing for explaining how the relative margin works with LoRA rank constraints.

**Meng et al. (2024).** *SimPO: Simple Preference Optimization with a Reference-Free Reward.* NeurIPS 2024.
- https://arxiv.org/abs/2405.14734
- **Why cited:** Length-normalized reward definition (Equation 4) and explicit margin γ as regularization. Load-bearing for explaining why SimPO removes DPO's bias toward short responses.

## Secondary reference

**Hong et al. (2024).** *ORPO: Monolithic Preference Optimization without Reference Model.* EMNLP 2024.
- https://arxiv.org/abs/2403.07691
- **Why cited:** Contrast case. ORPO combines SFT + preference in one loss; used as brief context for when to prefer ORPO over DPO/SimPO (retrain-from-base scenarios).

## Tools & systems inspected

**HuggingFace TRL `DPOTrainer`**
- https://huggingface.co/docs/trl/dpo_trainer
- **Why cited:** Documents `DPOTrainer` behavior and the PEFT/LoRA reference-model handling that makes `ref_model=None` a configuration detail to verify rather than assume.

**Python code snippet** (embedded in explainer)
- Concrete log-inspection function (`review_preference_run()`) that readers can adapt to their own TRL JSONL logs.
- Used as the hands-on pattern for comparing training margins against held-out metrics before declaring an objective winner.
