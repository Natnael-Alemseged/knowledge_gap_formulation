# Sources — Week 12

## Primary sources

**Willard, B. & Louf, R. (2023)**
*Efficient Guided Generation for Large Language Models*
arXiv:2307.09702
https://arxiv.org/abs/2307.09702

Published at EMNLP 2023 (arXiv preprint at the link above). The foundational paper on constrained decoding for LLMs. Introduces the finite-state machine approach to compiling JSON schemas into token-level vocabulary masks, achieving O(1) time per decoding step. Cited in the explainer as the mechanistic basis for why constrained decoding is a hard exclusion rather than a distribution shift.

---

**OpenAI (2024)**
*Structured Outputs — Platform Documentation*
https://platform.openai.com/docs/guides/structured-outputs

Documents OpenAI's implementation of token-level schema enforcement via `response_format: { type: "json_schema" }`. Cited for the claim that structured outputs are contracted to produce schema-valid output on every non-refused call, and for the qualifier that safety refusals can still return non-schema responses.

---

## Supporting references

**Outlines — dottxt-ai (2023–present)**
*Structured text generation library*
https://github.com/dottxt-ai/outlines

Reference implementation of the Willard & Louf constrained decoding approach. Mentioned in the explainer as the practical open-source entry point for grammar-guided sampling outside of provider APIs.

---

**llama.cpp — ggerganov (2023–present)**
*Grammar sampling via `--grammar-file`*
https://github.com/ggerganov/llama.cpp

Implements GBNF grammar-based constrained decoding for local open-weight models. Mentioned as the self-hosted equivalent to provider structured outputs.

---

## Tool / pattern used

The structured outputs migration in the grounding commit was implemented and verified using `openai` Python SDK (`client.beta.chat.completions.parse()`), with a Pydantic `BaseModel` schema as `response_format`. The before/after code in `grounding_commit.md` was run against the live judge endpoint to confirm the parse failure case and the schema-enforced success case. Outlines was reviewed for its FSM compilation approach but not deployed in production for this artifact; it is cited as the reference open-source implementation of Willard & Louf.

---

## Source quality notes

Both primary sources are direct: one conference paper (EMNLP 2023, arXiv preprint linked) establishing the mechanism, one official vendor documentation defining the contract. No secondary aggregators or forum posts were used. The supporting references are active open-source repositories maintained by the original authors or a large contributor community — not third-party summaries.
