## question.md

**Question:** My Week 10 sales agent routes the intent-classification
call through OpenRouter to `qwen/qwen3-235b-a22b`. In my CFO memo, I
wrote that inference costs are "optimized by provider caching," but I
never verified whether this model/provider combination actually exposes
prompt caching through OpenRouter. I also do not understand the exact
prefix condition required for a cache hit if caching is available:
would appending a new user message preserve a reusable static prefix,
or would interpolating dynamic fields like `{company_name}` into the
system prompt invalidate the prefix and force a full prefill again?
Knowing this would let me either redesign prompt construction to
benefit from caching or explain in my CFO memo why this path still pays
full prefill cost on each turn.

**Grounded artifact:** Week 10 CFO memo, cost projections section,
where I wrote that inference costs are "optimized by provider caching"
without verifying either (1) that OpenRouter prompt caching is
available for `qwen/qwen3-235b-a22b`, or (2) that my prompt structure
would qualify for a cache hit even if it were.

**What I currently believe:** I assumed OpenRouter automatically cached
repeated prompt prefixes across all models. I now suspect
`qwen/qwen3-235b-a22b` may not support prompt caching on OpenRouter at
all, and that my interpolated system prompt would break the reusable
prefix even if it did.

**What a closed gap looks like:** I can state definitively whether
prompt caching is available for my actual model/provider combination.
If yes, I can sketch the exact message-array structure that would
preserve a cacheable prefix and refactor `reply_intent.py` around it.
If no, I can delete the "optimized by caching" claim from my CFO memo
and replace it with the actual per-call cost model.

**What I will NOT cover:** Internal KV-cache mechanics inside
vLLM/TGI, or a broad provider survey beyond OpenRouter and the Qwen
hosting path relevant to this call.
