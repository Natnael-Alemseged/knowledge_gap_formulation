## thread.md

1. We added thousands of learned weights to an AI model.

   It ran at exactly the same speed as the original.

   But leave those weights in a slightly different form?
   Twice as slow.

   The difference is one function call. Here's what's happening inside:

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

5. Merged LoRA changes the values inside the weight matrices, but not
   the shape of the inference computation.

   Unmerged LoRA leaves an extra low-rank path alive during each forward
   pass — that is where the extra cost comes from.

6. Full explainer with benchmark code and the LoRA paper citation:
   https://dev.to/natnael_alemseged/why-merged-lora-barely-changes-inference-time-2mhj
