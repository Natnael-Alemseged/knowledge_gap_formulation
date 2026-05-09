# Week 12 Synthesis

**Natnael Alemseged — Forward-Deployed Engineer Trainee**

---

## The Eight Gaps Closed

### Gaps I named (as asker)

**Day 1 — Prompt caching availability and prefix conditions**
I claimed in my Week 10 CFO memo that inference costs were "optimized by provider caching" without ever verifying that `qwen/qwen3-235b-a22b` on OpenRouter supports caching, or that my interpolated system prompt would qualify for a cache hit. The gap closed when I understood that prefix caching requires a static, token-identical prefix — dynamic field interpolation like `{company_name}` invalidates the cache on every call — and that this specific model/provider path does not expose prompt caching at all, making the CFO claim factually wrong.

**Day 2 — How LLM function-calling works at the token level**
My sales agent in `openrouter_llm.py` sends vanilla chat completions with no `tools` parameter; the "tool choice" is deterministic Python scaffolding in `lead_orchestrator.py`. I described the system as an "agent that uses tools" without understanding what native tool calling actually does mechanically. The gap closed when I learned that the model generates a constrained token sequence (a special `<tool_call>` token followed by structured JSON) that the provider intercepts at the serving layer — not JSON the Python layer scrapes — and how this affects parsing reliability, latency, and refusal behavior.

**Day 3 — SimPO gradient mechanics and the ICP slice failure**
My LoRA judge failed on 67% of dual-segment ICP preference pairs after SimPO tuning, and my memo stated this without explaining why. I could not distinguish between five possible diagnoses: insufficient contrastive coverage, the SimPO margin objective itself, LoRA adapter capacity, beta/gamma settings, or a problem no trained judge could fix. The gap closed when I understood how SimPO's length-normalized margin and explicit γ interact with a base model prior that strongly favors the rejected output — and why targeted ICP contrast pairs are a principled fix rather than a hopeful data patch.

**Day 4 — Invalid bootstrap p-value in my evaluation script**
My benchmark script computed `P(mean_bootstrap ≤ 0)` from a bootstrap distribution centered at the observed lift and reported it as a one-sided p-value. The gap closed when I learned that a valid p-value must be computed under the null hypothesis — the bootstrap distribution must be centered at zero, not at the observed effect. The correct procedure for paired binary outcomes is McNemar's exact test: `binomtest(k=c, n=b+c, p=0.5, alternative='greater')`, which conditions only on the discordant pairs and produces a calibrated p-value. My invalid computation was 13× more significant than the correct result, not because the evidence was stronger but because the null was never imposed.

---

### Gaps I researched (as explainer)

**Day 1 — Why merged LoRA barely changes inference time**
The merge math `W_merged = W₀ + (α/r)BA` absorbs the adapter into the base weight matrix at load time. After merging, inference touches exactly the same number of parameters as the base model — there are no additional forward passes through adapter layers, no residual stream branching, and no memory overhead beyond the merged weights. I ran a three-way benchmark (base vs. unmerged vs. merged LoRA) on a T4 GPU and confirmed that merged inference latency is statistically indistinguishable from base model latency.

**Day 2 — The categorical distinction between instruction-following and constrained decoding**
"Return JSON only" does not force JSON — it shifts the probability distribution toward JSON-shaped tokens but leaves the model free to generate anything. Constrained decoding (the Willard & Louf FSM approach) compiles a JSON schema into a token-level vocabulary mask that is applied at every decoding step, making schema violations structurally impossible rather than just improbable. This is a hard constraint, not a soft one, and it is the mechanism behind OpenAI's structured outputs and the Outlines library.

**Day 3 — DPO vs SimPO: what your preference trainer is actually optimizing**
DPO defines an implicit reward model relative to a frozen reference model and penalizes KL divergence from it. SimPO removes the reference model entirely and uses a length-normalized margin with an explicit γ term. ORPO folds SFT and preference optimization into a single loss, making it the right choice when retraining from a base model. The three objectives penalize and reward different things, and choosing the wrong one for your data distribution and adapter capacity produces failures that look like data problems but are actually objective mismatches.

**Day 4 — Why paired bootstrap is necessary and when it stops helping**
Paired bootstrap is correct whenever the same tasks are evaluated under both systems (within-subject design) — resampling indices independently estimates uncertainty for a different experiment. But pairing narrows the CI only in proportion to the covariance between the two score vectors: `SE_paired = sqrt((Va + Vb - 2·Cov) / n)`. When the trained model passes tasks the baseline fails (large capability gap), the two score vectors decorrelate (r ≈ 0.17), and pairing provides almost no variance reduction — even though it remains the correct procedure by design.

---

## Question Quality Trajectory

Day 1 asked two sub-questions simultaneously (caching availability + prefix conditions) and described a belief rather than a mechanism. The artifact pointer was named but the grounding was vague: "I wrote X in my memo" rather than "this specific claim in this specific paragraph is what I cannot defend if pressed."

Day 2 was sharper. The question named the specific file and line where the gap was visible (`openrouter_llm.py`, lines 45–70), stated exactly what the current code does versus what native tool calling does, and named the concrete artifact that would change if the gap closed (blog post architecture section + payload comparison). The asker/explainer negotiation was also more productive because the question gave the explainer a clear target.

Day 3 was the most technically precise question of the week. It named five possible diagnoses for the ICP slice failure, explained why each implies a different next action, specified metric definitions and file:line pointers for every empirical claim, and stated what a satisfying answer looks like in three numbered outcomes. It also scoped explicitly what was out of scope. A senior engineer reading this question without context could reproduce the failure and evaluate a proposed fix.

Day 4 returned to concision — surgical rather than comprehensive. The broken line of code was named verbatim, the null hypothesis was stated precisely, and the four-property rubric was applied explicitly in a table. The question is shorter than Day 3's but arguably more diagnostic because it names exactly one thing that is wrong and exactly what would constitute a fix.

The trajectory: vague belief → artifact-grounded mechanism → multi-diagnosis triage with full empirical context → surgical single-line identification. The improvement was mostly in learning to ask "what specific claim in what specific artifact would change if I closed this gap?" rather than "what do I not understand about this topic?" Across the four completed days, the pattern became clear: the strongest questions were the ones that named the exact claim, file, metric, or behavior that would become more defensible after the research.

---

## The Most Surprising Thing I Learned

Three things surprised me this week, in increasing order of how much they changed how I think.

The least surprising in retrospect but most useful in practice: **research closed gaps in the opposite direction from expected.** On Day 1, my question was "how do I restructure my prompt to get cache hits on OpenRouter?" The answer was "your prompt is 600 tokens — below the minimum cacheable threshold — so the prefix structure is irrelevant." The CFO memo claim was wrong, but not for the reason I thought. The gap I named was about prefix conditions; the gap that actually closed was about prompt size. This happened again on Day 3: I expected the ICP slice failure to be an objective problem (SimPO couldn't overcome a strong base-model prior), but the evidence pointed to coverage starvation first. Both times, precise gap identification led to a finding that redirected the diagnosis. Vague questions would have produced answers that confirmed the wrong hypothesis.

More surprising: **the things I thought were on a reliability spectrum are actually categorically different mechanisms.** On Day 2 I learned that instruction-following ("return JSON only") and constrained decoding (logit masking against a grammar at every decode step) are not points on a reliability curve — they operate at different layers of the stack. No amount of prompt engineering reaches constrained decoding reliability because the two approaches are not comparable: one shifts a probability distribution, the other makes certain token sequences structurally impossible. The same applies to my "agent with tools" architecture claim: a system that prompt-engineers JSON extraction and one that sends a real `tools=[...]` schema are not more or less agentic — they run at categorically different inference layers. I had been treating architectural distinctions as degree differences, which made my portfolio language imprecise and my engineering decisions harder to defend.

Most surprising: **concordant pairs carry zero information.** On Day 4 I learned that in a paired binary evaluation, the tasks where both systems pass and the tasks where both systems fail contribute nothing to the question of which system is better. In my 48-task benchmark, 21 tasks were concordant — 15 both-pass, 6 both-fail. McNemar's test conditions only on the 27 discordant pairs. I had been computing significance over all 48 tasks, which felt more statistically powerful because the sample was larger. It is actually less correct, because the concordant tasks are not evidence about the relative advantage — they are evidence that the tasks exist. The discordant pairs are the only signal. Once I understood that, the 13× p-value inflation from my bootstrap script made immediate sense: I had been computing how stable the observed lift was, not how likely it was to appear by chance under the null. These are completely different questions.


---

## Canonical Reading List and Tool List

### Papers worth reading

**Hu et al. (2021). LoRA: Low-Rank Adaptation of Large Language Models.**
https://arxiv.org/abs/2106.09685
The merge math (`W_merged = W₀ + (α/r)BA`) and the zero-inference-overhead claim. Essential for anyone making latency or cost claims about LoRA-based systems.

**Willard & Louf (2023). Efficient Guided Generation for Large Language Models.**
https://arxiv.org/abs/2307.09702
The FSM approach to constrained decoding. Read this before choosing between prompt-based JSON extraction and schema-enforced structured outputs — it explains why the two are categorically different.

**Rafailov et al. (2023). Direct Preference Optimization.**
https://arxiv.org/abs/2305.18290
The implicit reward model equivalence and the KL regularization role. Load-bearing for understanding why DPO fails when the reference model's prior is far from the target distribution.

**Meng et al. (2024). SimPO: Simple Preference Optimization with a Reference-Free Reward.**
https://arxiv.org/abs/2405.14734
Length-normalized margin and explicit γ regularization. Read alongside the DPO paper to understand when removing the reference model helps and when it hurts.

**Hong et al. (2024). ORPO: Monolithic Preference Optimization without Reference Model.**
https://arxiv.org/abs/2403.07691
Combined SFT + preference in one loss. The right paper to read when deciding whether to fine-tune from a base model vs. a chat model.

**Efron & Hastie (2016). Computer Age Statistical Inference, Ch. 11.**
https://hastie.su.domains/CASI/
The definitive treatment of bootstrap confidence intervals. Chapter 11 covers the distinction between bootstrap CIs (valid) and bootstrap hypothesis tests (requires null-centered resampling). Free via Stanford.

**Dror et al. (2017). Replicability Analysis for Natural Language Processing.**
https://aclanthology.org/P19-1266/
Paired bootstrap and permutation tests for NLP evaluation. The canonical reference for anyone running A/B comparisons on matched task sets.

### Tools worth knowing

**HuggingFace PEFT — `merge_and_unload()`**
https://huggingface.co/docs/peft/conceptual_guides/lora
The practical entry point for LoRA merging. Read the merge math in the LoRA paper first, then use this to implement it.

**Outlines (dottxt-ai)**
https://github.com/dottxt-ai/outlines
Reference implementation of Willard & Louf constrained decoding. Use when you need schema-enforced generation outside of provider APIs.

**TRL CPOTrainer with `loss_type="simpo"`**
https://huggingface.co/docs/trl/dpo_trainer
The practical SimPO implementation. Note: `cpo_alpha` defaults to 1.0, which adds a chosen-output NLL regularizer — this is not paper-faithful standalone SimPO. Verify your config before claiming a clean SimPO run.

**`scipy.stats.binomtest`**
One line for McNemar's exact test on paired binary outcomes: `binomtest(k=c, n=b+c, p=0.5, alternative='greater')`. Use this instead of `P(mean_bootstrap ≤ 0)` whenever your evaluation design is within-subject and your outcomes are binary.
