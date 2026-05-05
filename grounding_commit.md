## grounding_commit.md

**Artifact:** Week 10 CFO memo, cost projections section
**Commit:** https://github.com/Natnael-Alemseged/tenacious-conversion-engine/commit/8f687d5c6501bd5fb90991d5dc69c6eccf274436

The memo previously claimed inference costs were "optimized by provider
caching" without verifying whether OpenRouter → qwen/qwen3-235b-a22b
actually exposes prompt caching or discounted cached-token billing. That
claim was removed and replaced with a defensible statement: the route has
not been verified for provider-side caching, so caching should not be
counted as a confirmed cost-saving mechanism. The updated memo names the
actual cost-control options available — reducing prompt length, using a
smaller model, replacing simple classifications with deterministic rules,
or switching to a provider/model combination with documented caching
support. The change was grounded in the explainer by Nebiyou Abebe, which
established that prompt caching requires explicit provider support and
that an unverified claim in a CFO memo is not a defensible engineering
position.
