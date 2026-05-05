## morning_call_summary.md

My original question conflated internal KV cache with API-level prompt caching and assumed OpenRouter automatically
cached all repeated prefixes; my partner's provider question exposed that I route to qwen/qwen3-235b-a22b and have not
verified whether this model even supports prompt caching, sharpening my question to a verifiable claim-or-delete
decision for my CFO memo. My partner's original question contained a false premise—that merging a LoRA adapter adds
parameters—and relied on single-run latency numbers; my Q2 about measurement variance forced an immediate edit to his
Week 11 memo, and my Q4 about merged vs. separate adapter reframed his question from a trivial "why isn't it slower" to
the deeper mechanism of why autoregressive decode is memory-bandwidth-bound and which levers actually reduce latency.
Both questions were committed final by the end of the call.