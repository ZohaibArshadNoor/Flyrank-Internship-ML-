# FL-01: AI Workflow Audit & Project Setup

**Author:** Zohaib Arshad Noor  
**Track:** AI Fluency & Machine Learning  
**Date:** August 2026  
**Repository:** [https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-](https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-)  

---

## 1. Weekly Workflow Audit (12 Recurring Tasks)

This audit maps real weekly recurring tasks across machine learning development, university coursework/research, and professional workflow against Ethan Mollick’s AI delegation framework:
- **Just me**: High-stakes, original reasoning, sensitive data, or high-touch personal judgment.
- **Delegate to AI with review**: Standardized drafting or scaffolding requiring human verification.
- **Collaborate with AI**: Interactive thought-partnership, exploratory design, or debugging.
- **Fully automate**: Deterministic, rule-bound, or scriptable tasks with zero ambiguity.

| # | Recurring Task | Life Domain | Classification | One-Line Rationale |
|:---:|:---|:---|:---:|:---|
| **1** | **Architectural System & Model Design** | ML Engineering | **Just me** | Requires holistic understanding of domain trade-offs, compute constraints, and business ethics that AI cannot reliably weigh. |
| **2** | **Code Review & Security Auditing** | Development | **Just me** | Ultimate accountability for regression bugs, secret leakage, and production safety rests strictly with human judgment. |
| **3** | **Personal Career Strategy & Milestone Review** | Personal / Career | **Just me** | Core values, risk appetite, and authentic life goals cannot be outsourced to statistical text generators. |
| **4** | **Daily Energy & Calendar Triage** | Productivity | **Just me** | Energy management and deep-work boundary protection require deliberate human self-awareness. |
| **5** | **Drafting Data Ingestion & Transformation Scripts** | ML Engineering | **Delegate to AI with review** | Pandas/DuckDB transformations follow standard patterns; human verifies edge cases and data types. |
| **6** | **Drafting Technical Documentation & API Specs** | Development | **Delegate to AI with review** | Fast boilerplate generation from code signatures; human reviews for clarity and precision. |
| **7** | **Research Paper Literature Scoping & Summarization** | Study / Research | **Delegate to AI with review** | Extracts core claims, methodology tables, and benchmark numbers; human reads primary text for deep nuances. |
| **8** | **Exploratory Debugging of Obscure Stack Traces** | Development | **Collaborate with AI** | Rapid brainstorming of root causes across library versions and environment quirks through interactive iteration. |
| **9** | **Drafting Test Cases & Unit Test Suites** | Development | **Collaborate with AI** | Identifies boundary conditions and mock fixtures; human verifies that critical business invariants are tested. |
| **10** | **Refining Professional Proposals & Cold Outreach** | Professional | **Collaborate with AI** | Tone polishing and conciseness adjustments while keeping authentic voice and specific context intact. |
| **11** | **Formatting & Linting Code on Git Commit** | Development | **Fully automate** | Pre-commit hooks (`black`, `flake8`, `isort`) enforce deterministic standards without cognitive effort. |
| **12** | **Weekly Status Summaries from Git Commits** | Project Mgmt | **Delegate to AI with review** | Aggregates commit logs into structured markdown bullet points for stakeholder updates. |

---

## 2. Three Target Tasks for FL-02 through FL-04

These three recurring workflows are selected for deep-dive optimization in subsequent AI fluency modules:

### Target Task 1: Research Paper Methodology Auditing (Target for FL-02)
* **Domain:** Study & Applied Machine Learning Research
* **Current Bottleneck:** Manually reading dense 20-page papers to locate the exact label origin, validation split design, and potential leakage takes 45+ minutes per paper.
* **Measurable "Done Well" Definition:**
  1. Identifies the exact dataset size, date windows, and target variable formulation within 5 minutes.
  2. Surfaces potential leakage vectors (e.g., temporal overlap, grouping violations) with specific section citations.
  3. Outputs a structured 1-page methodology critique comparing claimed metrics against naive baselines with zero hallucinated numbers.

---

### Target Task 2: Python / PyTorch Data Pipeline Scaffolding & Test Generation (Target for FL-03)
* **Domain:** Software & Machine Learning Development
* **Current Bottleneck:** Writing repetitive data loaders, categorical encoding routines, and pytest fixtures is tedious and prone to silent shape/type bugs.
* **Measurable "Done Well" Definition:**
  1. Produces clean, type-hinted, vectorized code complying with PEP 8.
  2. Generates comprehensive `pytest` suites covering edge cases (missing values, empty partitions, unexpected categorical levels).
  3. Code executes cleanly without syntax errors and requires $<10\%$ manual touch-up on first run.

---

### Target Task 3: Technical Briefing & Executive Summary Writing (Target for FL-04)
* **Domain:** Professional Communication & Research Dissemination
* **Current Bottleneck:** Translating complex model metrics (e.g., Precision@20 on grouped holdout splits) into clear, non-hyped executive summaries takes multiple draft cycles.
* **Measurable "Done Well" Definition:**
  1. Adheres strictly to the **Claim Ladder** (*observed*, *measured*, *directional*, *decision-support*).
  2. Produces a 3-tier audience breakdown: a 5-minute technical demo outline, a concise LinkedIn methodology cut, and a 3-sentence executive summary.
  3. Contains zero banned marketing clichés (*"revolutionary"*, *"guarantees 100% accuracy"*), with all claims grounded in empirical evidence.

---

## 3. Claude Project Custom Instructions

Below is the complete prompt configuration deployed in the **Claude Project Settings** (`Projects` → `New Project` → `Set custom instructions`):

```markdown
# Role & Persona
You are an expert Machine Learning Engineer, Research Collaborator, and Technical Writing Partner assisting Zohaib Arshad Noor. 

# Profile & Context
- Background: Computer Science & Machine Learning Research Intern working on tabular ML, search ranking, and production data pipelines.
- Current Goals: Building transparent, leakage-free ML models, mastering AI fluency workflows, and producing publication-grade technical research.
- Technical Stack: Python, PyTorch, Scikit-Learn, Pandas, DuckDB, Git, Markdown, LaTeX.

# Communication & Output Rules
1. Tone: Concise, analytical, evidence-grounded, and direct. Avoid conversational filler or flattery.
2. Code Standards: Clean, vectorized, type-annotated, PEP 8-compliant Python with explicit error handling.
3. Claim Discipline: Use safe, empirical claim language (observed, measured, associated, decision-support). Never assert causal claims without an experimental design.
4. Problem Solving: When debugging or designing pipelines, think systematically: state assumptions, inspect data grain/types, check for leakage, and prioritize readable code over unnecessary complexity.
```

---

## 4. Toolkit Setup & Verification Evidence

| Platform / Tool | Setup Status | Role in Workflow |
|---|:---:|---|
| **Claude.ai (Anthropic)** | **Configured & Active** | Primary technical reasoning, research paper synthesis, and structured code architecture via dedicated Claude Project. |
| **ChatGPT (OpenAI)** | **Configured & Active** | Comparative code reviews, cross-model verification, and secondary auditing. |
| **Anthropic Academy** | **Enrolled & Verified** | Enrolled in *AI Fluency: Framework & Foundations* (Module 1 completed: *Collaborating Effectively, Efficiently, Ethically, and Safely*). |

---

## 5. Self-Check & Submission Checklist

- [x] **10+ Genuine Tasks:** 12 authentic recurring tasks documented across ML engineering, development, study, and productivity.
- [x] **Clear Classification:** Every task classified into one of 4 framework categories with a one-line rationale.
- [x] **"Just Me" Tasks:** 4 tasks honestly designated as human-only with sound reasoning.
- [x] **3 Target Tasks with Success Rubrics:** Specific bottlenecks and measurable "Done Well" criteria defined for FL-02, FL-03, and FL-04.
- [x] **Claude Project Configured:** Custom instructions specifying role, context, tone, and claim discipline provided.
- [x] **Toolkit Evidenced:** Claude, ChatGPT, and Anthropic Academy enrollment documented.
