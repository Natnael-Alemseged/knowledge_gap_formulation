## signoff.md

**Gap-closure judgment: closed**

Before reading the explainer I had assumed OpenRouter automatically cached
repeated prefixes and that my CFO memo's "optimized by caching" claim was
basically correct. I now understand that the claim is unverifiable for my
actual route (OpenRouter → qwen/qwen3-235b-a22b) and should be removed
until billing logs or provider documentation confirm cache hits or
discounted cached-token billing. I also understand why interpolating
{company_name} into the system prompt can break prefix reuse if it changes
the cached prefix, and how to restructure the message array to keep static
instructions fixed while pushing dynamic fields into the user turn. The
corrected memo paragraph from the explainer is what I will use to update
the artifact.
