## sources.md

### Canonical sources

1. **Hu, E. J. et al. (2021). LoRA: Low-Rank Adaptation of Large Language Models.**
   [https://arxiv.org/abs/2106.09685](https://arxiv.org/abs/2106.09685)
   Primary source for the merge math (`W_merged = W₀ + (α/r)BA`) and the
   claim that merged adapters add no inference latency.

2. **HuggingFace PEFT — Conceptual guide to LoRA.**
   [https://huggingface.co/docs/peft/conceptual_guides/lora](https://huggingface.co/docs/peft/conceptual_guides/lora)
   Authoritative documentation for `merge_and_unload()`, adapter config,
   and the PEFT library used in the benchmark.

### Tool / pattern used

**PEFT 0.14.0 + Transformers 4.51.3**, run on a Colab T4 GPU.
Three-way benchmark: base model vs. unmerged LoRA vs. merged LoRA,
10 measured runs per condition, warmup discarded.
Adapter: `Natnaela/my-qwen-0.5b-lora` on base `Qwen/Qwen1.5-0.5B-Chat`.
Full benchmark code in [`instruction.md`](./instruction.md).
