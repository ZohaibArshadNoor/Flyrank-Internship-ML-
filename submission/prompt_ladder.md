# Prompt Ladder: From Lazy Query to Production ML Triage

**Author:** Zohaib Arshad Noor  
**Track:** AI Fluency & Prompt Engineering (Week 03: *The Prompt Ladder*)  
**Date:** September 2026  
**Repository:** [https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-](https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-)  

---

## Overview: The 5-Layer Prompt Progression

This document tracks the step-by-step engineering of a prompt for building an organic search content decay triage pipeline on a 30,000-page dataset. Each iteration adds **exactly one isolated layer** to address the primary failure mode of the previous output:

```
[ Run 0: Baseline ] ──> Vague query; outputs generic marketing fluff
        │  (+ Layer 1: Clear Goal & Task Framing)
[ Run 1: Goal Added ] ──> Generates code, but hallucinates non-existent columns
        │  (+ Layer 2: Real Context & Data Schema)
[ Run 2: Context Added ] ──> Uses real columns, but COMMITS MASSIVE TARGET LEAKAGE
        │  (+ Layer 3: Constraints & Leakage Boundaries)
[ Run 3: Constraints Added ] ──> Leakage-free, but produces an opaque, raw numeric score
        │  (+ Layer 4: Output Schema & Archetype Mapping)
[ Run 4: Schema Added ] ──> Actionable triage, but lacks verification & holdout metrics
        │  (+ Layer 5: Verification & Grouped Split Evaluation)
[ Run 5: Final Evaluation ] ──> Production-grade, leakage-free, verified triage pipeline
```

---

## Run 0: The Weak Baseline Prompt

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.
```

### Representative Output
> *"Here are several ways you can find bad content to refresh:*
> *1. Check pages with high bounce rates and low time-on-page in Google Analytics.*
> *2. Look for keywords where your rankings dropped from position 1 to page 2.*
> *3. Update old blog posts by adding new statistics, improving headings, and optimizing meta descriptions.*
> *4. Run an SEO audit using tools like Ahrefs or SEMrush to find broken links and thin content.*
> *Would you like me to give you a content refresh checklist?"*

---

## Run 1: Adding Layer 1 — Clear Goal & Task Framing

### The Added Layer
* **Layer Name:** Clear Goal & Task Framing
* **What was added:** Explicit instruction to write a Python/Pandas scoring script that ranks a tabular dataset of content items by traffic decline priority.

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.

Specifically, write a Python function using pandas that takes a DataFrame of published content items, computes a refresh priority score for each page, and returns the top 20 highest-priority pages to refresh.
```

### Representative Output
```python
def get_top_refresh_candidates(df):
    # Calculate refresh priority score
    df['refresh_score'] = (
        (1 - df['ctr']) * 0.4 + 
        df['bounce_rate'] * 0.3 + 
        df['traffic_drop_pct'] * 0.3
    )
    return df.sort_values('refresh_score', ascending=False).head(20)
```

### Four Notes on Run 1
1. **What changed in prompt:** Added a specific task goal (write a Python/Pandas scoring function returning the top 20 ranked pages).
2. **What actually improved in output:** Stopped outputting generic SEO advice bullet points; produced executable Python code.
3. **What still failed:** The model hallucinated non-existent columns (`bounce_rate`, `traffic_drop_pct`) that do not exist in our GSC/GA4 schema.
4. **What to try next:** Provide the exact column names, data types, and grain of the dataset.

---

## Run 2: Adding Layer 2 — Real Context & Data Schema

### The Added Layer
* **Layer Name:** Real Context & Data Schema
* **What was added:** Provided the actual dataset dimensions (30k rows, 32 clients) and the exact column names (`impressions_90d`, `clicks_90d`, `avg_position`, `days_since_last_update`, `trend_direction`, `trend_pct`, `impressions_last_30d`, `impressions_prev_30d`).

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.

Specifically, write a Python function using pandas that takes a DataFrame of published content items, computes a refresh priority score for each page, and returns the top 20 highest-priority pages to refresh.

Context:
- Dataset: 30,000 rows, one row per content item across 32 clients.
- Available columns: content_id, client_id, impressions_90d, clicks_90d, avg_position, ctr, days_since_last_update, content_age_days, trend_direction ('up', 'down', 'stable'), trend_pct, impressions_last_30d, impressions_prev_30d.
```

### Representative Output
```python
def get_top_refresh_candidates(df):
    # Flag declining pages using trend data
    is_declining = (df['trend_direction'] == 'down') | (df['trend_pct'] < -20)
    drop_magnitude = df['impressions_prev_30d'] - df['impressions_last_30d']
    
    # Priority based on trend decline and staleness
    df['priority_score'] = is_declining * drop_magnitude * (df['days_since_last_update'] / 30)
    return df.sort_values('priority_score', ascending=False)[['content_id', 'client_id', 'priority_score']].head(20)
```

### Four Notes on Run 2
1. **What changed in prompt:** Injected the exact dataset context, row grain, and available column schema.
2. **What actually improved in output:** The function now references real columns (`impressions_prev_30d`, `trend_direction`, `days_since_last_update`) instead of hallucinated metrics.
3. **What still failed (Honest Failure / "Made it worse" moment):** The model committed **massive circular target leakage**. It directly used `trend_direction`, `trend_pct`, and the 30-day comparison windows (`impressions_last_30d`, `impressions_prev_30d`) to score pages. In a real deployment, these sub-windows define the future outcome label itself.
4. **What to try next:** Add explicit negative constraints prohibiting label-derived inputs and future sub-windows.

---

## Run 3: Adding Layer 3 — Strict Constraints & Negative Rules

### The Added Layer
* **Layer Name:** Constraints & Negative Rules (Leakage Guardrails)
* **What was added:** Strict negative constraints forbidding `trend_direction`, `trend_pct`, and all 30-day sub-window metrics from being used as scoring inputs.

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.

Specifically, write a Python function using pandas that takes a DataFrame of published content items, computes a refresh priority score for each page, and returns the top 20 highest-priority pages to refresh.

Context:
- Dataset: 30,000 rows, one row per content item across 32 clients.
- Available columns: content_id, client_id, impressions_90d, clicks_90d, avg_position, ctr, days_since_last_update, content_age_days, trend_direction, trend_pct, impressions_last_30d, impressions_prev_30d.

Constraints & Negative Rules:
- ZERO TARGET LEAKAGE: You are strictly forbidden from using trend_direction, trend_pct, impressions_last_30d, or impressions_prev_30d in the scoring logic. Those columns represent the future evaluation window.
- Only use pre-decision observable signals: impressions_90d, avg_position, ctr, days_since_last_update, content_age_days.
- Do not use client_id or content_id as numerical features.
```

### Representative Output
```python
import numpy as np

def get_top_refresh_candidates(df):
    # Safe pre-decision signals only
    is_stale = (df['days_since_last_update'] >= 180).astype(int)
    is_visible = (df['impressions_90d'] >= 100).astype(int)
    pos_decay = np.clip(df['avg_position'] / 10.0, 1.0, 5.0)
    
    # Priority: Stale + Visible + Scaled by Search Exposure
    df['priority_score'] = is_stale * is_visible * np.log2(1 + df['impressions_90d']) * pos_decay
    return df.sort_values('priority_score', ascending=False)[['content_id', 'client_id', 'priority_score', 'impressions_90d', 'days_since_last_update']].head(20)
```

### Four Notes on Run 3
1. **What changed in prompt:** Enforced negative constraints banning label-derived fields and sub-window telemetry.
2. **What actually improved in output:** Successfully eradicated target leakage. The scoring formula relies purely on observable pre-decision historical signals.
3. **What still failed:** The output is an opaque, continuous float (`priority_score = 14.82`). An editorial team receiving this queue has no idea *what specific action* to take on each page (e.g. rewrite vs. title tag fix vs. consolidate).
4. **What to try next:** Define a structured output schema that maps items into concrete operational action archetypes with human-readable reason codes.

---

## Run 4: Adding Layer 4 — Output Schema & Archetype Mapping

### The Added Layer
* **Layer Name:** Output Schema & Action Archetype Mapping
* **What was added:** Specified a required output table schema with 4 discrete action labels (`REFRESH_AND_EXPAND`, `CTR_METADATA_OPTIMIZE`, `CONSOLIDATE_OR_RETIRE`, `MONITOR_MAINTAIN`) and explicit reason codes.

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.

Specifically, write a Python function using pandas that takes a DataFrame of published content items, computes a refresh priority score for each page, and returns the top 20 highest-priority pages to refresh.

Context:
- Dataset: 30,000 rows, one row per content item across 32 clients.
- Available columns: content_id, client_id, impressions_90d, clicks_90d, avg_position, ctr, days_since_last_update, content_age_days, trend_direction, trend_pct, impressions_last_30d, impressions_prev_30d.

Constraints & Negative Rules:
- ZERO TARGET LEAKAGE: You are strictly forbidden from using trend_direction, trend_pct, impressions_last_30d, or impressions_prev_30d in the scoring logic.
- Only use pre-decision observable signals: impressions_90d, avg_position, ctr, days_since_last_update, content_age_days.
- Do not use client_id or content_id as numerical features.

Output Schema & Action Mapping:
The returned DataFrame must include: [rank, content_id, client_id, playbook_action, reason_code, estimated_hours].
Map pages into these 4 archetypes:
1. REFRESH_AND_EXPAND (reason: 'stale_high_volume_decay', 3.5 hrs): days_since_last_update >= 180 and impressions_90d >= 500.
2. CTR_METADATA_OPTIMIZE (reason: 'strong_pos_weak_ctr', 0.75 hrs): avg_position <= 20 and ctr < 0.5 and impressions_90d >= 100.
3. CONSOLIDATE_OR_RETIRE (reason: 'low_traffic_staleness', 1.5 hrs): days_since_last_update >= 180 and impressions_90d < 100.
4. MONITOR_MAINTAIN (reason: 'stable_healthy_asset', 0.0 hrs): all other items.
```

### Representative Output
```python
def build_action_playbook_queue(df):
    def classify(row):
        stale = row['days_since_last_update'] >= 180
        imp = row['impressions_90d']
        pos = row['avg_position']
        ctr = row['ctr']
        
        if stale and imp >= 500:
            return 'REFRESH_AND_EXPAND', 'stale_high_volume_decay', 3.5, 1
        elif 0 < pos <= 20 and ctr < 0.5 and imp >= 100:
            return 'CTR_METADATA_OPTIMIZE', 'strong_pos_weak_ctr', 0.75, 2
        elif stale and imp < 100:
            return 'CONSOLIDATE_OR_RETIRE', 'low_traffic_staleness', 1.5, 3
        else:
            return 'MONITOR_MAINTAIN', 'stable_healthy_asset', 0.0, 4

    results = df.apply(classify, axis=1)
    df['playbook_action'] = [r[0] for r in results]
    df['reason_code'] = [r[1] for r in results]
    df['estimated_hours'] = [r[2] for r in results]
    df['priority_rank'] = [r[3] for r in results]
    
    queue = df.sort_values(by=['priority_rank', 'impressions_90d'], ascending=[True, False]).reset_index(drop=True)
    queue['rank'] = queue.index + 1
    return queue[['rank', 'content_id', 'client_id', 'playbook_action', 'reason_code', 'estimated_hours', 'impressions_90d', 'days_since_last_update']].head(20)
```

### Four Notes on Run 4
1. **What changed in prompt:** Specified a structured output schema, 4 discrete action archetypes, reason codes, and estimated workload hours.
2. **What actually improved in output:** The output is now an actionable decision-support triage queue that editors can immediately execute without deciphering raw floats.
3. **What still failed:** The code lacks an evaluation framework. We have no empirical proof that this ranking strategy actually captures declining articles better than a naive random baseline on unseen clients.
4. **What to try next:** Add a rigorous validation protocol requiring Grouped Cross-Validation by client and Precision@K reporting.

---

## Run 5: Adding Layer 5 — Verification & Grouped Holdout Evaluation

### The Added Layer
* **Layer Name:** Verification & Grouped Holdout Evaluation Protocol
* **What was added:** Required the script to include an out-of-sample evaluation step using `GroupShuffleSplit` on `client_id` (0% client overlap), calculating Precision@20 vs. the naive base rate.

### The Prompt
```text
Help me analyze this content dataset and find bad pages to refresh.

Specifically, write a Python function using pandas and scikit-learn that takes a DataFrame of published content items, computes a refresh priority score for each page, and returns the top 20 highest-priority pages to refresh.

Context:
- Dataset: 30,000 rows, one row per content item across 32 clients.
- Available columns: content_id, client_id, impressions_90d, clicks_90d, avg_position, ctr, days_since_last_update, content_age_days, trend_direction, trend_pct, impressions_last_30d, impressions_prev_30d.

Constraints & Negative Rules:
- ZERO TARGET LEAKAGE: You are strictly forbidden from using trend_direction, trend_pct, impressions_last_30d, or impressions_prev_30d in the scoring logic.
- Only use pre-decision observable signals: impressions_90d, avg_position, ctr, days_since_last_update, content_age_days.
- Do not use client_id or content_id as numerical features.

Output Schema & Action Mapping:
The returned DataFrame must include: [rank, content_id, client_id, playbook_action, reason_code, estimated_hours].
Map pages into 4 archetypes: REFRESH_AND_EXPAND (stale_high_volume_decay), CTR_METADATA_OPTIMIZE (strong_pos_weak_ctr), CONSOLIDATE_OR_RETIRE (low_traffic_staleness), MONITOR_MAINTAIN (stable_healthy_asset).

Verification & Evaluation Requirements:
1. Include an evaluation harness that defines y = (trend_direction == 'down').astype(int).
2. Split the dataset using GroupShuffleSplit(n_splits=1, test_size=0.25, random_state=42) grouped by client_id.
3. Compute and print Precision@20 on the held-out test clients alongside the naive test base rate to prove generalization.
```

### Representative Output
```python
import pandas as pd
import numpy as np
from sklearn.model_selection import GroupShuffleSplit

def run_content_triage_pipeline(df):
    # 1. Define ground-truth target (evaluation only)
    y = df['trend_direction'].str.lower().eq('down').astype(int).values
    groups = df['client_id'].values
    
    # 2. Honest Grouped Split (Zero client overlap)
    gss = GroupShuffleSplit(n_splits=1, test_size=0.25, random_state=42)
    train_idx, test_idx = next(gss.split(df, y, groups))
    test_df = df.iloc[test_idx].copy()
    y_test = y[test_idx]
    
    # 3. Leakage-Free Baseline Scoring Formula
    is_stale = (test_df['days_since_last_update'] >= 180).astype(int)
    is_visible = (test_df['impressions_90d'] >= 100).astype(int)
    scores = (is_stale * is_visible * np.log2(1 + test_df['impressions_90d'])).values
    
    # 4. Evaluation: Precision@20 on Unseen Clients
    order = np.argsort(-scores)
    p20 = y_test[order[:20]].mean()
    base_rate = y_test.mean()
    
    print("=== EVALUATION ON UNSEEN CLIENT HOLDOUT ===")
    print(f"Test Clients: {test_df['client_id'].nunique()} | Test Rows: {len(test_df):,}")
    print(f"Observed Precision@20: {p20:.3f} vs. Naive Base Rate: {base_rate:.3f} (+{p20 - base_rate:.3f} pp lift)")
    
    # 5. Build Content Action Playbook Queue
    def classify(row):
        stale = row['days_since_last_update'] >= 180
        imp = row['impressions_90d']
        pos = row['avg_position']
        ctr = row['ctr']
        if stale and imp >= 500:
            return 'REFRESH_AND_EXPAND', 'stale_high_volume_decay', 3.5, 1
        elif 0 < pos <= 20 and ctr < 0.5 and imp >= 100:
            return 'CTR_METADATA_OPTIMIZE', 'strong_pos_weak_ctr', 0.75, 2
        elif stale and imp < 100:
            return 'CONSOLIDATE_OR_RETIRE', 'low_traffic_staleness', 1.5, 3
        else:
            return 'MONITOR_MAINTAIN', 'stable_healthy_asset', 0.0, 4

    res = test_df.apply(classify, axis=1)
    test_df['playbook_action'] = [r[0] for r in res]
    test_df['reason_code'] = [r[1] for r in res]
    test_df['estimated_hours'] = [r[2] for r in res]
    test_df['priority_rank'] = [r[3] for r in res]
    
    queue = test_df.sort_values(by=['priority_rank', 'impressions_90d'], ascending=[True, False]).reset_index(drop=True)
    queue['rank'] = queue.index + 1
    return queue[['rank', 'content_id', 'client_id', 'playbook_action', 'reason_code', 'estimated_hours', 'impressions_90d', 'days_since_last_update']].head(20)
```

### Four Notes on Run 5
1. **What changed in prompt:** Added the verification protocol requiring grouped train/test isolation by client and Precision@20 vs. base rate reporting.
2. **What actually improved in output:** The script is now a self-contained scientific pipeline that verifies generalization on unseen client domains before generating the operational queue.
3. **What still failed:** None — all failure modes (vagueness, column hallucinations, target leakage, opaque floats, lack of evaluation) are resolved.
4. **What to try next:** Extract the complete specification into a modular, reusable prompt template for peer adoption.

---

## 6. The Final Clean & Reusable Prompt

Below is the standalone, production-ready prompt that any ML engineer on the track can execute out-of-the-box:

```markdown
# TASK: Build a Leakage-Free Content Refresh Triage Pipeline

## 1. Goal
Write a self-contained Python script using pandas, numpy, and scikit-learn that:
1. Evaluates a supervised content-refresh priority ranking rule on held-out client portfolios.
2. Generates an operational top-20 Content Action Playbook queue for human editorial triage.

## 2. Dataset Context
- Input: Tabular DataFrame representing 30,000 published content items across 32 pseudonymized client domains.
- Pre-decision observable columns: `content_id`, `client_id`, `impressions_90d`, `clicks_90d`, `avg_position`, `ctr`, `days_since_last_update`, `content_age_days`.
- Evaluation-only target columns: `trend_direction` ('down', 'stable', 'up'), `trend_pct`, `impressions_last_30d`, `impressions_prev_30d`.

## 3. Strict Constraints & Leakage Guardrails
- ZERO LEAKAGE: Do NOT use `trend_direction`, `trend_pct`, or any 30-day sub-window metrics in the scoring formula. They define the evaluation label and are strictly forbidden as features.
- Identifiers (`client_id`, `content_id`) are for grouping and joins only, never as numerical features.
- Validation must use `GroupShuffleSplit` on `client_id` (test_size=0.25, random_state=42) to ensure 0% client overlap between train and test splits.

## 4. Operational Playbook Output Schema
Map ranked items into these 4 discrete action archetypes:
- `REFRESH_AND_EXPAND` (reason: `stale_high_volume_decay`, 3.5 hrs): `days_since_last_update >= 180` and `impressions_90d >= 500`.
- `CTR_METADATA_OPTIMIZE` (reason: `strong_pos_weak_ctr`, 0.75 hrs): `avg_position <= 20`, `ctr < 0.5%`, and `impressions_90d >= 100`.
- `CONSOLIDATE_OR_RETIRE` (reason: `low_traffic_staleness`, 1.5 hrs): `days_since_last_update >= 180` and `impressions_90d < 100`.
- `MONITOR_MAINTAIN` (reason: `stable_healthy_asset`, 0.0 hrs): all other items.

## 5. Required Verification Output
Print the evaluation summary showing:
- Number of test clients and test rows.
- Observed Precision@20 on the test holdout.
- Naive test base rate (`y.mean()`) and percentage-point lift.
- The formatted top-20 ranked queue containing `[rank, content_id, client_id, playbook_action, reason_code, estimated_hours, impressions_90d, days_since_last_update]`.
```

---

## 7. Pass / Revise Verification Checklist

- [x] **Six Runs Total:** Baseline (Run 0) plus 5 progressive versions documented.
- [x] **One Named Layer per Version:** (1) Goal, (2) Schema/Context, (3) Negative Constraints, (4) Output Schema, (5) Grouped Verification.
- [x] **Output-Focused Notes:** Notes describe specific behavioral changes in generated code and data outputs, not just prompt additions.
- [x] **Honest Failure Moment Included:** Run 2 honestly documents how adding context triggered severe circular target leakage, requiring negative constraints in Run 3.
- [x] **Final Prompt Reusable:** Standalone prompt template ready for any engineer to execute without prior context.
