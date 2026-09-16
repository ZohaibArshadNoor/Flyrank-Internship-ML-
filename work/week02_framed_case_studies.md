# Week 02: Framed Case Studies & Portfolio Voice

**Author:** Zohaib Arshad Noor  
**Track:** AI Fluency & Portfolio Build (Week 02: *Frame It as Cases*)  
**Date:** September 2026  
**Repository:** [https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-](https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-)  
**Deployed Research Paper:** [https://zohaibarshadnoor.github.io/Flyrank-Internship-ML-/](https://zohaibarshadnoor.github.io/Flyrank-Internship-ML-/)  

---

## 1. The Voice Card

> ### **Voice Card:**  
> **"Direct, analytical, plain-spoken, evidence-grounded, zero-buzzwords."** (6 words)

### Standing Instruction for Claude Project:
```markdown
# Voice & Style Guardrails
- Speak directly, plainly, and concisely. No throat-clearing, no preamble.
- Banish resume clichés: "results-driven", "passionate", "cutting-edge", "leveraged", "pioneered".
- Every claim must state the metric, the split, the base rate, and the sample size.
- Maintain a skeptical engineering posture: celebrate finding leakage and bounds as much as high scores.
```

---

## 2. Flagship Case Study: Organic Search Decay & Content-Refresh Prioritisation

### Beat 1: The Problem
Across enterprise digital publications, content follows a silent decay curve: ranking positions slip, CTR erodes, and organic search traffic evaporates over 6–12 months. For a portfolio of **30,000 pages across 32 client domains**, manually auditing every URL is impossible. 

Editorial bandwidth is structurally fixed at **10 to 20 comprehensive article updates per week**. The decision is an asymmetric ranking problem:
- **False Positive Cost:** An editor spends 4 hours rewriting an article whose traffic was already stable, yielding near-zero marginal recovery.
- **False Negative Cost (Critical):** A high-exposure Page 1 asset in active decline is ignored, slipping to Page 3 and losing thousands of monthly organic impressions.

---

### Beat 2: What I Did (and the Decisions I Made)
1. **Established a Transparent Heuristic Baseline First:** Before training any ML model, I encoded the standard industry rule: $\text{Priority} = \mathbb{I}(\text{stale} \ge 180\text{d}) \times \mathbb{I}(\text{impressions} \ge 100) \times \log_2(1 + \text{impressions})$. This achieved 55.0% Precision@20 on held-out data.
2. **Hunted and Eliminated Target Leakage:** Early gradient boosting experiments produced an unrealistic `0.999 ROC-AUC`. I audited the features and found that sub-window columns (`impressions_last_30d`, `impressions_prev_30d`) were the exact mathematical inputs used to define `trend_pct` (the target label). I strictly purged all sub-window metrics, label-derived flags, and client IDs from the feature set.
3. **Enforced Zero-Client-Overlap Grouped Validation:** Standard random splits let models memorize client-specific domain authority. I implemented a **Grouped Shuffle Split by `client_id`** (24 clients train / 8 clients test), forcing the model to prove generalization on completely unseen client architectures.
4. **Trained an Interpretable Gradient Boosting Ranker:** Fitted a Histogram-based Gradient Boosting model on clean 90-day pre-decision aggregates (`days_with_impressions`, `avg_position`, `content_age_days`, `ctr`).

---

### Beat 3: What Came of It
- **Observed +35 pp Precision Lift:** On the held-out test split of 8 unseen clients (7,115 pages), the model achieved **90.0% Precision@20** (18/20 true declines correctly identified) compared to the 55.0% rule baseline and a 51.7% naive random base rate.
- **Engineered an Operational Content Action Playbook:** Translated probability scores into 4 distinct operational archetypes with standardized reason codes (`REFRESH_AND_EXPAND`, `CTR_METADATA_OPTIMIZE`, `CONSOLIDATE_OR_RETIRE`, `MONITOR_MAINTAIN`) and explicit review effort allocations.
- **Built Strict Human-in-the-Loop Guardrails:** Established a firm No-Go policy prohibiting automated 301 redirects, direct-to-CMS auto-publishing, and unreviewed changes to YMYL/checkout URLs.
- **Published Live Open-Source Research Paper:** Deployed the full methodology, interactive charts, and serialized metric receipts at [zohaibarshadnoor.github.io/Flyrank-Internship-ML-](https://zohaibarshadnoor.github.io/Flyrank-Internship-ML-/).

---

## 3. About / Bio Copy

```markdown
### About
I am a Machine Learning Engineer focused on tabular modeling, search telemetry, and production data integrity. 

My engineering philosophy rests on three non-negotiables:
1. **Baselines First:** Never train a complex model until a transparent heuristic rule is benchmarked on the exact same split and metric.
2. **Ruthless Leakage Auditing:** If a model scores near 1.0 AUC, assume leakage until proven otherwise. Isolate sub-windows and enforce grouped splits by domain.
3. **Decision-Support Over Automation:** Machine learning should triage high-dimensional data for human experts with transparent reason codes, not unilaterally execute irreversible actions.
```

---

## 4. Contact & Call-to-Action Copy

```markdown
### Let's Talk
I am looking for Applied Machine Learning and Search Science roles at teams working on messy, high-scale tabular data.

- **Primary Action:** [Schedule a 20-minute Technical Chat (Direct Calendar Link) ↗]
- **Direct Email:** [zohaibarshadnoor@gmail.com]
- **Code & Receipts:** [github.com/ZohaibArshadNoor/Flyrank-Internship-ML- ↗]
- **Interactive Research Paper:** [zohaibarshadnoor.github.io/Flyrank-Internship-ML- ↗]
```

---

## 5. Before vs. After Copy Edit (Voice Card in Practice)

This side-by-side demonstrates how applying the Voice Card (*"Direct, analytical, plain-spoken, evidence-grounded, zero-buzzwords"*) strips out generic AI bloat in favor of evidence:

| Aspect | Generic AI Draft (Before) | Hard-Edited Voice (After) |
|---|---|---|
| **Headline Claim** | *"I leveraged state-of-the-art cutting-edge gradient boosting algorithms and comprehensive feature engineering to revolutionize content marketing ROI, successfully predicting search engine trends with unprecedented accuracy."* | *"I built a leakage-free Gradient Boosting ranking pipeline on 30,000 Search Console pages that achieved 90.0% Precision@20 (+35 pp over baseline) on unseen client domains, prioritizing high-exposure content for human refresh."* |
| **Why it Changed** | Stuffed with empty buzzwords (*"leveraged"*, *"state-of-the-art"*, *"revolutionize"*, *"unprecedented"*) and overclaims causal prediction of Google's algorithm. | States the exact model family, dataset scale, evaluation metric, holdout type, baseline lift, and operational decision without hype. |

---

## 6. Pass / Revise Verification Checklist

- [x] **Framed Case Study Included:** Complete 3-beat narrative covering problem, decisions/leakage hunt, and verified results.
- [x] **Voice Card Defined:** 6-word card (*Direct, analytical, plain-spoken, evidence-grounded, zero-buzzwords*) with Claude instructions.
- [x] **Specific & Unique:** Could only describe this 30,000-page Search Console grouped-split capstone project.
- [x] **Targeted Audience & Action:** Speaks directly to an Applied ML Engineering Director with a single-click call-to-action.
- [x] **Concrete Before/After Edit:** Clear side-by-side comparison demonstrating the elimination of generic AI fluff.
