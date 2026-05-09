# Canonical List — Week 12

*Annotated papers, tools, and patterns worth every Forward-Deployed Engineer reading.*

---

## Training and post-training mechanics

**Hu, E. J. et al. (2021). LoRA: Low-Rank Adaptation of Large Language Models. arXiv:2106.09685**
https://arxiv.org/abs/2106.09685
[paper]
The foundational paper on parameter-efficient fine-tuning. Names the merge math (`W_merged = W₀ + (α/r)BA`) and proves that merged adapters add zero inference latency. Reach for it when a client asks why you chose LoRA over full fine-tuning, or when you need to defend a rank choice — the paper's rank ablations are the canonical starting point.

---

**Rafailov, R. et al. (2023). Direct Preference Optimization: Your Language Model is Secretly a Reward Model. NeurIPS 2023. arXiv:2305.18290**
https://arxiv.org/abs/2305.18290
[paper]
Derives the DPO loss from the RLHF objective and shows the model is implicitly a reward model. Equation 7 explains why DPO can fail when the base model's prior strongly favors the rejected output — the gradient signal is too weak to overcome it. Reach for it whenever a preference-tuned judge underperforms on a specific slice despite correct training setup.

---

**Meng, Y. et al. (2024). SimPO: Simple Preference Optimization with a Reference-Free Reward. NeurIPS 2024. arXiv:2405.14734**
https://arxiv.org/abs/2405.14734
[paper]
Replaces DPO's per-token KL regularizer with a length-normalized reward and explicit margin γ, removing the need for a reference model. Equation 4 is the load-bearing formula. Reach for it when training a judge on a small dataset without reference-model memory overhead, or when diagnosing why DPO biases toward short outputs.

---

**Hong, J. et al. (2024). ORPO: Monolithic Preference Optimization without Reference Model. EMNLP 2024. arXiv:2403.07691**
https://arxiv.org/abs/2403.07691
[paper]
Combines SFT and preference optimization in a single loss with no reference model. The right choice when fine-tuning from a raw base (not instruction-tuned) checkpoint. Reach for it as the comparison case when deciding between DPO, SimPO, and ORPO — instruction-tuned base → DPO/SimPO; raw base → ORPO.

---

**HuggingFace TRL — DPOTrainer / CPOTrainer documentation**
https://huggingface.co/docs/trl/dpo_trainer
[docs]
Authoritative reference for TRL's DPO and CPO implementations (SimPO runs via `loss_type="simpo"` in CPOTrainer). Critical detail: `cpo_alpha` defaults to `1.0`, adding a chosen-output NLL regularizer even in SimPO mode — many practitioners run unintentional hybrid objectives without knowing it. Audit this before every preference training run.

---

**HuggingFace PEFT — Conceptual guide to LoRA**
https://huggingface.co/docs/peft/conceptual_guides/lora
[docs]
Documents `merge_and_unload()`, adapter config, and target module selection. The go-to reference when a LoRA adapter behaves differently merged vs. unmerged, or when a client asks what "rank 16, alpha 32" means in practice.

---

## Inference-time mechanics and structured output

**Willard, B. & Louf, R. (2023). Efficient Guided Generation for Large Language Models. arXiv:2307.09702**
https://arxiv.org/abs/2307.09702
[paper]
Introduces the FSM approach to constrained decoding: compiling a JSON schema into per-token vocabulary masks at O(1) per step. This is the mechanistic basis for why structured outputs are a hard exclusion (not a distribution shift) and cannot produce schema-invalid tokens on a non-refused call. Reach for it when a client asks "why not just prompt for JSON?" — this paper is the answer.

---

**OpenAI Structured Outputs — Platform Documentation**
https://platform.openai.com/docs/guides/structured-outputs
[docs]
Documents the API contract for `response_format: { type: "json_schema" }`. Key guarantee: schema-valid output on every non-refused call. Key exception: safety refusals can still return non-schema responses. Reach for it when designing a production judge or parser that needs to distinguish parse failure from valid non-answers.

---

**Outlines — dottxt-ai**
https://github.com/dottxt-ai/outlines
[tool]
Open-source reference implementation of the Willard & Louf constrained decoding approach for local open-weight models. The practical entry point when you need structured output guarantees without a provider API — self-hosted or air-gapped deployments.

---

**llama.cpp — GBNF grammar sampling**
https://github.com/ggerganov/llama.cpp
[tool]
Implements grammar-based constrained decoding via `--grammar-file` for local quantized models. The self-hosted equivalent of provider structured outputs for Qwen, Llama, or Mistral.

---

## Evaluation and statistics

**Efron, B. & Hastie, T. (2016). Computer Age Statistical Inference, Ch. 11. Stanford University Press.**
https://hastie.su.domains/CASI/
[paper]
The authoritative treatment of bootstrap CIs. Chapter 11 establishes the distinction between bootstrap CIs (valid for estimating sampling variability) and bootstrap hypothesis tests (which require resampling under the null). Reach for it whenever reporting a bootstrap CI on an LLM eval — the chapter prevents the most common significance-testing error in the field.

---

**Dror, R. et al. (2017). Replicability Analysis for Natural Language Processing: Testing Significance with Multiple Datasets. TACL 2017.**
https://aclanthology.org/P19-1266/
[paper]
Canonical NLP reference for paired bootstrap and permutation tests in model evaluation. Directly applicable to any within-subject benchmark where two systems are evaluated on the same task set. Reach for it when writing up evaluation results — this is what reviewers will cite if your significance test is wrong.

---

## Engineering patterns

**`scipy.stats.binomtest` — McNemar's exact test for paired binary data**
https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.binomtest.html
[tool]
One-line implementation of McNemar's exact test: `binomtest(k=c, n=b+c, p=0.5, alternative='greater')`. The correct null-hypothesis significance test for paired binary evaluation (pass/fail per task, same tasks for both systems). Use this instead of a bootstrap p-value whenever you report significance on a matched benchmark.

---

**NumPy `default_rng` paired bootstrap pattern**
https://numpy.org/doc/stable/reference/random/generator.html
[tool]
The correct pattern for paired bootstrap CIs: resample a single index array and apply it to both score vectors. Resampling each vector independently breaks the paired structure and estimates uncertainty for a different (unrun) experiment. Two lines that prevent a common evaluation error.

---

