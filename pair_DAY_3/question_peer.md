# Research question — Day 3 (training and post-training mechanics)

**To:** Natnael Alemseged  
**From:** Hiwot Beyene  
**Purpose:** My Day 3 gap selection, grounded in my Week 11 `tenacious-bench` artifacts.

---

## 1) Topic for the day

**Training and post-training mechanics**

Subtopics in scope from the Day 3 slate:

- What LoRA actually adapts; why low rank works; what changes as rank increases.
- Gradient mechanics of DPO vs SimPO vs ORPO.
- Reward model overoptimization and the role of KL regularization.
- Instruction-following vs reasoning data competition during post-training.

---

## 2) Gap candidates from my Week 11 implementation (ranked)

| Rank | Candidate gap | Why it matters in my repo |
|---|---|---|
| 1 | **Objective/mechanics mismatch:** my code path trains with **TRL `DPOTrainer`** (`training/preference_lora_train.py`), while my methodology narrative argues **SimPO-first** (`docs/methodology_rationale.md`). I cannot currently explain, mechanistically, what gradients and regularization behavior changed (or did not) because of this mismatch. | This weakens the causal claim behind Week 11 outcomes in `reports/model_card.md` and `reports/FINAL_WEEK11_MEMO_REPORT.md`. |
| 2 | **LoRA rank sensitivity is unproven:** I used LoRA (`r=16`, alpha 32, dropout 0.05) but cannot defend why this rank is the right bottleneck for a 0.5B base in my preference setup. | If rank is wrong, “trained vs baseline” deltas may reflect under/over-capacity rather than objective quality. |
| 3 | **Overoptimization risk is under-instrumented:** `rewards/margins` and train loss improve, but I do not have a clean “reward hacking” check beyond small held-out accuracy. | Production deployment risk: critic could optimize judge surface while harming buyer-safe behavior. |
| 4 | **Instruction vs reasoning mixture is implicit:** preference prompts are policy-heavy, but I do not quantify whether this suppresses deeper reasoning behaviors needed for nuanced weak-signal cases. | Could explain why gains are small and unstable on `n=22` held-out pairs. |
| 5 | **DPO/SimPO/ORPO comparison is narrative, not experimental:** I cite all three in memos, but no controlled same-data ablation exists. | Makes algorithm claims non-falsifiable in portfolio review. |

**Selected gap for today:** **Rank #1** (objective/mechanics mismatch) with LoRA-rank and overoptimization as adjacent supporting concepts.

---

## 3) The gap in one paragraph

My Week 11 artifacts present a Path B preference-tuning story, but I cannot currently defend the training objective at gradient level across my own files. `training/preference_lora_train.py` uses `DPOTrainer(ref_model=None)` with LoRA adapters, while `docs/methodology_rationale.md` frames SimPO as the primary objective and ORPO as fallback. Because my held-out lift is small and uncertain (`reports/model_card.md`: trained 22.73% vs baseline 18.18%, CI touches zero), I need a mechanistic answer to whether my observed behavior is consistent with DPO-style implicit reward shaping, SimPO-style reference-free length-normalized margins, or simply LoRA capacity/regularization effects. Without that, I cannot tell whether to modify objective, LoRA rank, or data before Day 4+.

---

## 4) Final research question (precise + concise)

In my current Week 11 setup (`training/preference_lora_train.py`: TRL `DPOTrainer` + LoRA on `Qwen/Qwen2.5-0.5B-Instruct`, pairs from `training_data/preferences.jsonl`), what is the smallest decisive ablation that tells me whether to stay on DPO or switch to SimPO/ORPO?

Please answer four points only:

1. **Gradient-level difference:** In this exact setup, what changes in the update signal between DPO vs SimPO vs ORPO on the same `(prompt, chosen, rejected)` tuples?
2. **Overoptimization diagnostics:** Which 1-2 metrics from my existing logs/artifacts best detect “reward-model/judge-surface optimization” vs real policy improvement?
3. **LoRA rank interaction:** For preference tuning here, what failure pattern should I expect from rank-too-low vs rank-too-high, and which observable should I watch first?
4. **Decision rule:** What concrete objective x rank matrix (minimum runs) and stop/continue criterion would make the DPO-vs-SimPO/ORPO choice evidence-based this week?

---

## 5) Artifact anchors for your explainer

- `tenacious-bench/training/preference_lora_train.py` (current trainer/objective and LoRA config)
- `tenacious-bench/docs/methodology_rationale.md` (SimPO-first narrative and paper grounding)
- `tenacious-bench/reports/model_card.md` (held-out deltas, training metrics, `cost_pareto`)
- `tenacious-bench/reports/FINAL_WEEK11_MEMO_REPORT.md` (CFO-facing claims and caveats)
- `tenacious-bench/training_data/README.md` + `training_data/preferences.jsonl` (pair construction assumptions)

---

## 6) What “gap closed” means tonight

I will consider this gap closed if I can:

1. Explain my **actual** current objective and its gradient signal in plain language tied to my code.
2. State one falsifiable reason to prefer DPO vs SimPO vs ORPO for my data regime.
3. Propose one concrete Day 3/4 ablation plan (small, realistic) with a clear stop/continue rule.

---

## 7) Clarification replies to my peer (Natnael)

Below are your full clarification questions, with my direct answers based on my Week 11 implementation in [`tenacious-bench`](https://github.com/Hiwot-Beyene/tenacious-bench).

### Q1) Priority across the four asks

**Your question:**  
"You bundle gradient mechanics, KL/overoptimization signals, LoRA rank, and an ablation plan. Which one is the must-go-deep mechanism, and which can I treat as brief context? My instinct is gradient mechanics (DPO vs SimPO) is the load-bearing piece — confirm?"

**My answer:**  
Yes, confirm. **Must-go-deep = gradient mechanics (DPO vs SimPO in my exact setup).**  
Please treat the others as supporting context in this order:

1. **Overoptimization/KL diagnostics** (brief but concrete; which metrics I should trust from my existing logs),
2. **LoRA rank** (short but operational: how rank can fake objective conclusions),
3. **Ablation plan** (concise decision rule at the end).

If space is tight, spend most depth on the **objective-mechanics mismatch** between my actual code (`DPOTrainer`) and my SimPO-first narrative.

### Q2) ORPO scope

**Your question:**  
"Your code path is DPO, your narrative is SimPO-first, and ORPO is mentioned as fallback. Should I compare all three, or focus on DPO vs SimPO with ORPO as a one-paragraph pointer? (Keeps the explainer tight on the actual tension in your repo.)"

**My answer:**  
Please **focus on DPO vs SimPO** as the main comparison.  
Use **ORPO as a short fallback pointer** (one compact paragraph): when I would consider it, and what would trigger that fallback.  

Reason: the real implementation tension in my repo is DPO code path vs SimPO framing; making ORPO co-equal would dilute the most important mismatch I need to resolve tonight.

### Q3) “Falsifiable reason” format

**Your question:**  
"\"Falsifiable reason\" — what kind? You want one falsifiable reason to prefer DPO vs SimPO for your data regime. Do you mean a theoretical signal (e.g., \"if your pair lengths vary by >X, SimPO wins\") or an experimental one (e.g., \"run this comparison; if held-out metric Y doesn't improve, switch\")? This shapes what I can realistically deliver in ~800 words."

**My answer:**  
I want an **experimental falsifiable reason** (primary), not only a theoretical heuristic.  
Please give me a testable statement in this shape:

- "If we run A/B with fixed data/model/seed controls and metric Y does not improve (or degrades beyond threshold T), we do **not** prefer objective X."

You can include one short theoretical intuition, but the deliverable I need is a **decision-ready experiment rule** that I can execute this week.

### Q4) LoRA rank scope

**Your question:**  
"LoRA rank: in or out? You ranked it #2 in your own triage but folded it into the main question. Should I treat rank as part of the gradient story (how rank bottlenecks preference learning) or scope it out with a note that it's the next gap after objective? I want to avoid going shallow on two mechanisms when I could go deep on one."

**My answer:**  
Keep LoRA rank **in scope but lightweight**: treat it as a **confounder on objective interpretation**, not a second deep dive.

What I need from rank is:

- one clear explanation of **rank-too-low vs rank-too-high** failure signatures in my preference setting,
- one observable to watch first (from my existing training/eval artifacts),
- and one small rank choice in the ablation matrix (e.g., current rank vs one higher rank).

So: go deep on **objective mechanics**, and include rank only to prevent me from drawing the wrong conclusion about DPO vs SimPO.
