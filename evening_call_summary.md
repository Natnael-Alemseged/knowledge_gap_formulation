## evening_call_summary.md

The asker's main feedback was that the initial draft explained the merge
math correctly but did not make clear why the memory-bandwidth bottleneck
matters specifically during decode rather than prefill — the distinction
between the two phases was named but not connected to why it is the reason
merged LoRA cannot save time. The writer added the prefill/decode split
section and the matrix-vector multiply explanation to close that gap. The
asker also flagged that the original single-run Week 11 numbers were
presented as if they proved something, so the writer reframed them as
likely noise and added the controlled three-way benchmark with mean and
standard deviation across 10 runs. On revision, the asker confirmed the
gap closed: they now understand that merged LoRA changes weight values but
not the inference graph, and that the actual latency levers are
quantization, batching, speculative decoding, and model size — not the
adapter merge operation.
