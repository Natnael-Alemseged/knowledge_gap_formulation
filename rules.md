# Rules File - Week 12 Knowledge Gap Formulation

This file records the rules used to turn Week 12 research into defensible portfolio edits. It is written for a reviewer checking whether the submission connects engineering decisions, research artifacts, and concrete revisions to the Week 10 and Week 11 portfolio.

## Submission Scope

This repository documents four completed paired research days:

| Day | Topic | Portfolio connection |
|---|---|---|
| Day 1 | Prompt caching and LoRA inference mechanics | Week 10 cost memo and inference-cost claims |
| Day 2 | Structured output and tool-use internals | Week 10 agent reliability, JSON parsing, fallback logging, and model-card language |
| Day 3 | Preference optimization and SimPO/DPO mechanics | Week 11 judge-training diagnosis and ICP failure slice |
| Day 4 | Paired evaluation statistics and McNemar testing | Week 11 benchmark scripts, README, CFO memo, and significance claims |

## Research Rules

1. Each day must start from a specific gap in an existing Week 10 or Week 11 artifact, not from a generic topic.
2. The question must name the artifact whose quality changes if the gap is closed.
3. The explainer must rely on primary or canonical sources where possible: papers, official documentation, or runnable tooling.
4. Each day must include at least one hands-on artifact: code, benchmark instructions, log-inspection pattern, statistical test, or concrete before/after implementation.
5. The final write-up must distinguish what was learned from what was changed. A useful explanation is not enough unless it produces a portfolio edit or a documented reason for the edit.

## Engineering Decision Rules

1. Do not make cost, latency, caching, reliability, or statistical-significance claims unless the mechanism is verified or the uncertainty is explicitly named.
2. Treat LLM structured output as a trust boundary. Prompt instructions such as "return JSON only" are soft constraints; production code must either use schema-constrained decoding or raise/log parse failures clearly.
3. Treat benchmark significance tests as part of the system contract. Confidence intervals and p-values must answer the question they claim to answer and must match the experimental design.
4. When a trained judge fails on a named slice, document the mechanism and the next testable intervention. Do not collapse targeted-data fixes, objective changes, and capacity changes into "add more data."
5. When the portfolio language describes an architecture, it must match the actual serving path: deterministic scaffolding, provider-native tool calling, constrained decoding, and post-hoc parsing are different mechanisms.

## Portfolio Edit Rules

Each grounding commit must include:

1. The artifact edited.
2. The claim or behavior that was wrong, weak, or undefended.
3. The new defensible claim or implementation behavior.
4. The research source or experiment that justified the change.
5. A pointer to the commit, diff, or exact before/after snippet.

## Completed Portfolio Edits

| Day | Edit rule applied | Result |
|---|---|---|
| Day 1 | Unverified cost claims must be removed or qualified. | Removed the unverified "optimized by provider caching" claim from the Week 10 CFO memo and replaced it with defensible cost-control options. |
| Day 2 | Structured-output failures must be observable and schema guarantees must be described accurately. | Documented `_safe_parse_json` before/after behavior, structured-output migration, fallback behavior, and model-card language changes. |
| Day 3 | Training-failure claims must name the slice, mechanism, and next intervention. | Updated the Week 11 bench memo to name the ICP failure slice, wrong log-probability ordering, targeted contrast-pair rationale, and 80% re-check gate. |
| Day 4 | Statistical claims must use a null distribution that matches the paired binary design. | Replaced the invalid bootstrap p-value with McNemar's exact test and propagated the corrected significance claim across Week 11 artifacts. |

## Evidence Rules

1. `README.md` is the submission index: it should let a reviewer find every daily artifact quickly.
2. `synthesis.md` is the reflection layer: it should explain the gaps closed and the trajectory of question quality without overclaiming uncompleted days.
3. `portfolio_update.md` is the hiring-manager layer: it should explain why the grounding commits improve the portfolio.
4. `canonical_list.md` is the source layer: it should collect the papers, tools, and implementation patterns that made the edits defensible.
5. Daily folders are the audit trail: `question.md`, `sources.md`, `explainer.md`, `thread.md`, `morning_call_summary.md`, `evening_call_summary.md`, `signoff.md`, and `grounding_commit.md` should connect the research loop end to end.
