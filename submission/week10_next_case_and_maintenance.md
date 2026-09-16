# Week 10: Portfolio Maintenance Protocol & Next Case Study Roadmap

**Author:** Zohaib Arshad Noor  
**Track:** AI Fluency & Portfolio Build (Week 10: *Send the Link & Keep It Alive*)  
**Date:** September 2026  
**Repository:** [https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-](https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-)  
**Deployed Research Paper:** [https://zohaibarshadnoor.github.io/Flyrank-Internship-ML-/](https://zohaibarshadnoor.github.io/Flyrank-Internship-ML-/)  

---

## 1. "How to Add the Next Case" — Standard Operating Procedure

To ensure the portfolio evolves from a single capstone showcase into an ongoing career asset, adding a new project follows a deterministic 4-step pipeline:

```
[ Step 1: 10-Minute Claude Project Interview ] ──> Drafts the 3-beat narrative in voice
        │
[ Step 2: Generate & Save Visual Receipts ] ──> Exports 2 figures to docs/img/ & metrics JSON
        │
[ Step 3: Insert Case Card in docs/index.html ] ──> Updates Featured Work section & links
        │
[ Step 4: Git Push to Main ] ──> Auto-deploys to GitHub Pages in <2 minutes
```

### The 3-Beat Content Shape (Reused from Week 2):
1. **Beat 1: The Problem & Operational Context**  
   - Who is the user, what is the dataset scale, what decision is being made, and what is the asymmetric cost of false positives vs. false negatives?
2. **Beat 2: What I Did (and Key Architectural Decisions)**  
   - Transparent heuristic baseline first, leakage audit and exclusions, validation split design (grouped/time-aware), and model choice.
3. **Beat 3: What Came of It (Empirical Receipts & Playbook)**  
   - Verified metric lift over baseline on held-out test data, operational action playbook with reason codes, and links to open-source code receipts.

---

## 2. Named Next Piece of Real Work

* **Project Title:** **Real-Time Search Intent Disambiguation & Click Velocity Classification Pipeline**
* **Technical Scope:**  
  - Developing an asynchronous embedding and gradient-boosted ranking pipeline that classifies search query intent shifts (Informational $\rightarrow$ Commercial $\rightarrow$ Transactional) and predicts 30-day click velocity drops on enterprise search logs.
* **Stack & Data:** Python, DuckDB, `FastEmbed` / Sentence-Transformers, Scikit-Learn, LightGBM, and Google Search Console query telemetry (100k+ keyword-URL pairs).
* **Target Claim & Metric:**  
  - Achieves $>85\%$ Precision@50 in detecting intent drift on cross-domain holdouts, powering automated title/meta refresh recommendations.

---

## 3. Concrete Reminder & Calendar Schedule

To prevent portfolio stagnation, an active calendar reminder and recurring maintenance trigger have been set:

* **Next Portfolio Sprint Event:** **October 15, 2026 at 10:00 AM**
* **Calendar Entry Title:** `🚀 Portfolio Ingestion Sprint: Add Search Intent & Click Velocity Case Study`
* **Cadence:** Recurring bi-monthly review on the 15th of every even month.
* **Execution Checklist in Calendar Invite:**
  - [ ] Run benchmark pipeline and export metrics JSON to `work/outputs/`.
  - [ ] Open existing Claude Project (`Portfolio & ML Research Showcase Tutor`) and run the 3-beat drafting prompt.
  - [ ] Place 2 new charts into `docs/img/`.
  - [ ] Add the case card to `docs/index.html`, commit, and push to `main`.

---

## 4. Preserved Claude Project Continuity

The **`Portfolio & ML Research Showcase Tutor`** Claude Project remains permanently configured with our standing identity kit:

* **Voice Card Standing Rule:** *"Direct, analytical, plain-spoken, evidence-grounded, zero-buzzwords."*
* **Claim Discipline Guardrail:** Enforces the Claim Ladder (*observed*, *measured*, *directional*, *decision-support*) and prohibits unearned causal claims without randomized control designs.
* **Turnaround Speed:** Because Claude already holds the complete profile, target persona (Engineering Directors / Applied ML Leads), and style constraints, drafting the next case study requires only a single 15-minute conversation rather than rebuilding prompt instructions from scratch:

```text
# Copy-Paste Prompt for the Next Case Study:
I have finished a new project: Real-Time Search Intent & Click Velocity Classification.
Here are my raw notes on the problem, architecture, grouped holdout split, and Precision@50 numbers:
[Paste raw notes]

Interview me with 3 sharp questions to clarify the trade-offs, then draft the 3-beat case study using our standing Voice Card and Claim Ladder rules.
```

---

## 5. Submission Checklist & Self-Check

- [x] **Concrete SOP Documented:** Clear 4-step sequence and 3-beat structure defined for future updates.
- [x] **Specific Next Work Named:** Real upcoming project (*Search Intent & Click Velocity Classification*) specified with technical stack and target metrics.
- [x] **Calendar Reminder Set:** Real calendar date (October 15, 2026) and recurring bi-monthly review schedule recorded.
- [x] **Claude Project Preserved:** Standing voice card, target audience, and 10-minute intake prompt documented for rapid future authoring.
