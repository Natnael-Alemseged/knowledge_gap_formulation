## thread.md

1. Why did a merged LoRA version of Qwen1.5-0.5B-Chat take almost the
   same inference time as the bare base model — even though it has extra
   trained weights baked in?

2. The key is the difference between merged and unmerged LoRA.
   Unmerged: `y = W₀x + BAx` — the adapter runs as a separate branch
   every forward pass.
   Merged: `y = W_merged x` — the adapter no longer exists at inference
   time. It was folded into the weights before the first token.

3. That means merged LoRA is not "base model plus adapter" at generation
   time. It is just the base model with different weight values. Same
   tensor shapes. Same memory traffic. Same decode loop.

4. A three-way benchmark on the same Colab T4 confirms it
   (10 runs each, warmup discarded):

   Base:     27 ms ± 1 ms
   Unmerged: 58 ms ± 5 ms  ← 2.15× slower
   Merged:   26 ms ± 1 ms  ← within noise of base

5. The real speed levers are the ones that reduce memory traffic or
   decode steps: quantization, batching, speculative decoding, or a
   smaller model. Merged LoRA touches none of these — which is exactly
   why it shows up as a near-zero latency difference.

6. Full explainer with benchmark code and the LoRA paper citation:
   https://dev.to/natnael_alemseged/why-merged-lora-barely-changes-inference-time-2mhj
