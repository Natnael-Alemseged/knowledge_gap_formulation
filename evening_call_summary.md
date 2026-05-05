## evening_call_summary.md

My partner confirmed the central confusion was "why extra learned weights
don't slow inference," so I revised the explainer to make the
merged-vs-unmerged distinction load-bearing: after `merge_and_unload()`,
LoRA stops being an extra computation and becomes a single dense weight
matrix. He pushed me to name the real bottleneck (decode as
memory-bandwidth-bound) rather than leaving it as "GPU overhead," and to
separate prefill vs decode explicitly. He also flagged that the original
timing gap could be noise, so I added a controlled three-way benchmark
(base vs unmerged vs merged, repeated runs with warmup discarded) to make
the mechanism visible and verifiable. We agreed the revised blog and
thread now answer the question cleanly and are ready to ship.
