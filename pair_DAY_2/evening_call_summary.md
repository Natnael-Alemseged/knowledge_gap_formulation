# Evening Call Summary — Week 12

**Partners:** Natnael Alemseged & Kirubel Gashaw
**Date:** May 6, 2026

---

## What we set out to do

Review and sharpen the peer question assigned to us for Week 12, write the explainer answering it, and produce all supporting artifacts for the portfolio commit.

---

## How the session went

The scheduled morning call shifted to Slack when Kirubel's audio dropped early. Rather than wait for a reschedule, we made a judgment call: Natnael committed to assumptions about Kirubel's question, built all artifacts from that reading, and sent the finished explainer link for Kirubel to review asynchronously. This was a protocol deviation — the rubric specifies real-time voice calls — and it is documented as such. The practical outcome was good; the process gap is real.

**Note on file pairing:** `question.md` in this folder is Natnael's own question (tool-calling mechanism, assigned to Kirubel to answer). `explainer.md` is the explainer Natnael wrote for Kirubel's question (structured output reliability). In the paired-research structure these are intentionally about different topics — they are not expected to match.

---

## Peer question (Kirubel's — as Natnael interpreted it)

**As received:**
> "In my Week 11 pipeline, I told an LLM judge to 'return JSON only' and it usually did. But when it didn't, my pipeline broke silently. Does telling a model to return JSON actually *force* it to, or does it just make JSON more likely? What is the real mechanism, and what does it mean for any system that acts on structured LLM output?"

**Gaps identified (Natnael, working solo):**
- Conflates instruction-following (soft mechanism) with constrained decoding (hard mechanism) without naming either
- Doesn't specify whether the asker wants the mechanism explanation or the engineering fix — both are implied
- "Broke silently" is the most important detail but is treated as background, not the core problem

**How Natnael structured the explainer:** Used a two-axis framing (mechanism vs system design) to address both implied questions. Led with the failure mode ("broke silently") as the concrete anchor, then named both mechanisms, showed code, connected to tool calling as an adjacent concept.

---

## Artifacts produced

| Artifact | Status | Location |
|---|---|---|
| `explainer.md` | Published live | https://dev.to/natnael_alemseged/return-json-only-doesnt-force-json-heres-what-actually-forces-it-9pn |
| `thread.md` | Ready to post | `/thread.md` |
| `grounding_commit.md` | Complete | `/grounding_commit.md` |

---

## Key decisions made

**Constrained decoding as the core mechanism.** The explainer centres on Willard & Louf (2023) as the canonical source. Instruction-following vs constrained decoding is framed as categorical, not quantitative — a distribution shift vs a hard exclusion.

**Three precision fixes applied after review:**
- Temperature 0 handling: "every token is a draw" was incorrect for greedy decoding — corrected to distinguish argmax (temp=0) from sampling (temp>0), with the key point that deterministic decoding reduces but does not eliminate the risk
- OpenAI guarantee: "guarantee on every call" was overclaimed — corrected to "contracted to produce schema-valid output on every non-refused call" with an explicit note on safety refusals
- Adjacent concept added: a paragraph connecting constrained decoding to provider-native tool calling (same inference-layer mechanism), which ties Kirubel's question to the wider landscape

**"Back to my system" section added.** Grounds the explainer in the real artifact (`ledger/agents/credit_analysis_agent.py`, `_safe_parse_json`) with before/after code and a representative failing input.

---

## What still needs doing

- [ ] Post the X thread (file is ready, link is live)
- [ ] Follow up with Kirubel — send the published explainer link and confirm gap closed

---

## Lesson from this session

Committing to assumptions and building beats waiting. The two-axis framing (mechanism vs system design) and the grounding section came from working through the question alone rather than waiting for direction. When a peer is quiet, move — document the assumption, build the artifact, show your work.
