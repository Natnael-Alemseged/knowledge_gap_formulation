## explainer.md

# Why Merged LoRA Barely Changes Inference Time

**The question:** In a Week 11 ablation, a merged LoRA version of
Qwen1.5-0.5B-Chat took 14,228 ms per task, while the bare base model took
14,045 ms. That 183 ms gap is only about 1.3%. Why doesn't merging in
extra trained weights make inference slower? And if the adapter is not
the thing driving latency, what actually is?

The short answer is: **once LoRA is merged, the model is no longer doing
"base model plus adapter" at inference time. It is just doing the base
model computation with a different set of weight values.** The tensor
shapes do not change, the number of layers does not change, and the
number of bytes that must be moved for each generated token is almost
the same. On modern GPUs, that last point matters most.

One caution before explaining the mechanism: with only one timing run
per system on a shared Colab T4, you cannot prove that 183 ms is
"real." A 1.3% gap is **plausibly noise**, not evidence that merged LoRA
adds meaningful latency. The mechanism below explains why we should
expect the difference to be near zero, and the controlled benchmark
below confirms it directly.

## What merged LoRA changes, and what it does not

Before merging, a LoRA-adapted linear layer is effectively:

`y = W₀x + (α/r)BAx`

where `W₀` is the original weight matrix and `BA` is the low-rank LoRA
update. In that form, inference really does include extra operations:
you still apply the base matrix, and you also apply the low-rank update.

After `merge_and_unload()`, those two pieces are combined ahead of time:

`W_merged = W₀ + (α/r)BA`

Now inference uses:

`y = W_merged x`

That matters because the model no longer carries separate adapter
modules through the forward pass. At generation time, there is no "plus
adapter" branch left to execute. The model performs the same sequence of
layer operations it did before, using weight tensors with the same
shapes and usually the same dtype as the base model.

So the key intuition is not "LoRA weights are free." The key intuition
is: **merged LoRA stops being a separate computation.**

This is the core mechanism described in the original LoRA paper (Hu et al.,
2021, [arXiv 2106.09685](https://arxiv.org/abs/2106.09685)), which notes
that merging incurs no additional inference latency because the adapter
is folded into the original weights before any forward pass runs.

## Where token-generation time actually goes

To explain why this often makes almost no latency difference, we need to
separate two phases of inference:

**Prefill.** The model processes the input prompt and builds the KV
cache. This phase can use larger matrix-matrix style operations because
many prompt tokens are processed together.

**Decode.** The model generates one new token at a time, reusing the KV
cache and running a forward pass for just the next token.

When people talk about autoregressive generation being slow, they are
usually talking about **decode**, not prefill. Decode is where latency
becomes dominated by repeated small forward passes over the model's
weights.

At each layer during decode, the core linear operation is effectively a
matrix-vector multiply: a hidden-state vector for one token multiplied by
a weight matrix. That is a bad regime for GPUs because the computation
per byte of memory moved is low. The GPU spends much of its time waiting
for weights to be read from memory rather than saturating its compute
units with arithmetic.

That is why merged LoRA usually does not show up in decode latency. If
`W_merged` has the same shape and dtype as `W₀`, then each token still
requires moving essentially the same amount of model data through memory.
The values inside the matrix changed, but the amount of work the GPU
must schedule and the amount of memory it must read are almost the same.

This is the load-bearing mechanism behind your peer's result: **for
merged LoRA, the expensive part is still streaming the same-sized weight
tensors and running the same decode loop, not "carrying extra learned
knowledge."**

## The controlled benchmark

To go beyond the single-run Week 11 numbers, the following three-way
benchmark was run on the same Colab T4 — base model vs. unmerged adapter
vs. merged adapter — with 10 measured runs per condition (first run
discarded as warmup) and identical generation settings throughout.

**Setup:** `Qwen/Qwen1.5-0.5B-Chat` base model, adapter
`Natnaela/my-qwen-0.5b-lora`, `MAX_NEW_TOKENS=64`, `do_sample=False`,
`float16`, PEFT 0.14.0.

| Condition | Mean latency (s) | Std dev (s) | Runs |
|-----------|:---:|:---:|:---:|
| Base | 0.027 | 0.001 | 10 |
| Unmerged LoRA | 0.058 | 0.005 | 10 |
| Merged LoRA | 0.026 | 0.001 | 10 |

The pattern matches the prediction exactly:

- **Merged ≈ base** (26.5 ms vs 27.1 ms). The standard deviations
  overlap completely. After merging, the forward pass is identical in
  structure to the base model.
- **Unmerged is 2.15× slower than base** (58.3 ms vs 27.1 ms). The
  extra low-rank matrix multiplications `BAx` run on every forward pass,
  and at the small batch sizes used in decode they add real cost.

This also recontextualises the original 14,228 ms vs 14,045 ms gap from
Week 11. Those were full agent task timings — prompt processing, tool
calls, multi-step generation — not isolated generation latency. The
183 ms difference there was likely noise or tool-call variance, not
evidence that merging adds cost.

The benchmark code used to produce these numbers is reproduced in
[`instruction.md`](./instruction.md) and can be rerun directly in Colab.

## Why the benchmark result is believable

If the adapter had been left unmerged, then a slowdown would be easier
to explain: extra low-rank multiplications would be happening during the
forward pass. But once the adapter is baked in, the inference path looks
like a normal dense model with modified parameter values. In that case,
we expect:

1. Similar per-token decode cost, because tensor shapes are unchanged.
2. Similar memory traffic, because the same kinds of weights still need
   to be read.
3. Small observed differences to be dominated by run-to-run noise unless
   the benchmark is repeated many times under controlled conditions.

The benchmark confirms all three. The right claim for the memo is not
"merged LoRA is mathematically free." The right claim is narrower and
more accurate: **merged LoRA does not materially change the inference
graph or memory footprint per token, so it usually does not materially
change latency.**

## A simple analogy

Imagine two books with the same number of pages, same paper size, and
same binding weight, but different text printed inside. If your job is
to carry one book from one room to another, the time is determined
mostly by the size and weight of the book, not by which words are on the
pages.

Merged LoRA is similar. You are still carrying a model of essentially
the same size through the same inference pipeline. The content of the
weights changed, but the "shape of the object" the GPU has to move
through memory did not.

## What would actually make the model faster

This also answers the peer's last question: if merged LoRA is not the
latency lever, what is?

The biggest levers are the ones that change memory traffic, parallelism,
or the number of decode steps:

1. **Quantization.** Lower-precision weights reduce how many bytes must
   be moved per token.
2. **Batching.** More concurrent tokens/sequences can increase hardware
   utilization and improve throughput.
3. **Speculative decoding.** A draft model can reduce how often the full
   model must do slow one-token-at-a-time work.
4. **A smaller model or different architecture.** Fewer or smaller
   weight tensors mean less work and less data movement.

These are meaningful speed levers because they attack the actual
bottleneck. Merged LoRA does not.

## What the Week 11 memo should say

The clean takeaway for the evaluation report is:

> The merged LoRA adapter did not materially change latency because
> `merge_and_unload()` folds the low-rank update into the base weights
> ahead of inference. After merging, generation runs the same model
> structure with the same tensor shapes, so decode cost remains dominated
> by loading and applying base-sized weight matrices rather than by any
> separate adapter computation. A controlled three-way benchmark (base vs.
> unmerged vs. merged, 10 runs each on a T4) confirms this: merged and
> base land within noise of each other (27 ms vs. 26 ms), while unmerged
> is 2.15× slower (58 ms) due to the extra low-rank path still running
> at inference time.

## Sources

- Hu et al. (2021). *LoRA: Low-Rank Adaptation of Large Language Models.*
  [arXiv:2106.09685](https://arxiv.org/abs/2106.09685)
- HuggingFace PEFT documentation — Conceptual guide to LoRA, including
  the `merge_and_unload()` API.
  [huggingface.co/docs/peft/conceptual_guides/lora](https://huggingface.co/docs/peft/conceptual_guides/lora)
- Benchmark tool: PEFT 0.14.0 + Transformers 4.51.3, run on Colab T4.
  Full code in [`instruction.md`](./instruction.md).
