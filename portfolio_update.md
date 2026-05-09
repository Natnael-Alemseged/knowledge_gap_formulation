# Portfolio Update — Week 12

*How four grounding commits improve the Week 10 and 11 portfolio.*
*Written for an FDE hiring manager evaluating production AI system candidates.*

---

## What changed and why it matters

Before Week 12, my Week 10 and 11 artifacts contained claims I could not defend under technical scrutiny: a cost projection that assumed caching without verifying it, an architecture description that misrepresented how tool-calling actually works, a training failure diagnosis that named the symptom but not the mechanism, and a significance claim built on a statistically invalid p-value. None of these were visible as errors from the outside — the systems ran, the numbers looked reasonable, and the memos read confidently. Week 12 forced me to find and fix the places where my writing outran my understanding. The four grounding commits below are the evidence that the completed gaps are closed.

---

## The four commits

| Day | Commit | Artifact touched | What became defensible |
|-----|--------|-----------------|------------------------|
| 1 | [8f687d5](https://github.com/Natnael-Alemseged/tenacious-conversion-engine/commit/8f687d5c6501bd5fb90991d5dc69c6eccf274436) | Week 10 CFO memo — cost projections | Removed "optimized by provider caching" claim. Can now state whether OpenRouter + Qwen exposes prompt caching and what prefix structure would qualify — and can defend why the claim was wrong rather than just deleting it. |
| 2 | [4a7d41b](https://github.com/Natnael-Alemseged/tenacious-conversion-engine/commit/4a7d41bdb05f28885753291397a6f58bffc4ae33) | Week 10 agent — `openrouter_llm.py`, parse/fallback logic, tests | JSON parse failures and invalid model labels are now observable in production logs. Fallback results distinguish `parse_failed` from normal `other`. PII no longer logged. Tests cover the new behavior. Can now explain the categorical difference between deterministic orchestration and provider-native tool calling — and defend why the current approach was the right choice for this system's reliability requirements. |
| 3 | [565d98e](https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/565d98e) | Week 11 bench memo — paragraph 32, ICP failure diagnosis | Memo now names the actual mechanism: SimPO's gradient cannot overcome the base model's prior on dual-segment ICP pairs with insufficient contrastive data. Adds targeted contrast-pair fix rationale and 80% re-check gate as deployment criterion. Can now distinguish "add targeted ICP pairs" (principled fix) from "add data and hope" for a specific failure slice. |
| 4 | [c841f47](https://github.com/Natnael-Alemseged/SalesConversion-Bench/commit/c841f47631bfa3df8d02e8bcc24a73b83adaac53) | Week 11 eval — `paired_bootstrap_delta_a.py`, `run_ablations.py`, CFO memo, README, evidence graph, ablation summary | Replaced `P(mean_bootstrap ≤ 0)` with McNemar's exact test (`binomtest`) across every artifact that cited the significance claim. Can now explain why the original number was 13× too small, what null distribution is correct for paired binary outcomes, and why the `training_wins` verdict is robust to the method choice in this data. |

---

## What the portfolio demonstrates now

The portfolio now shows three things it could not show before Week 12. First, it shows that I can find errors in my own work under time pressure — not errors caught by tests or reviewers, but the subtler class of errors where the system runs correctly and the claim is wrong. Second, it shows that I can close gaps with primary sources rather than secondary summaries: each fix traces to a paper, a specification, or a runnable experiment, not to intuition or a blog post. Third, it shows that the Week 10 and 11 systems are defensible at the mechanism level — I can explain why the architecture choices were made, what the training failure actually means, and what the evaluation numbers actually prove. That combination — shipping a system, finding what you got wrong, and fixing it with the depth an FDE engagement requires — is the competency this portfolio is designed to demonstrate.
