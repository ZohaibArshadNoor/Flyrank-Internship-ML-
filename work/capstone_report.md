# Capstone Report — Content-Refresh Prioritisation via Supervised Ranking

- **Author:** Zohaib Arshad Noor
- **Lane:** Content-Refresh Prioritisation
- **Repo:** https://github.com/ZohaibArshadNoor/Flyrank-Internship-ML-
- **Date:** August 2026

---

## 0. Abstract

Maintaining organic search visibility across large content portfolios is challenging as published articles quietly suffer traffic decay from changing algorithms, competitor activity, and factual obsolescence. Using a dataset of 30,000 pseudonymized pages across 32 clients from the FlyRank ML Internship dataset, we evaluated supervised ranking approaches to prioritize decaying articles for editorial refresh. We established a strict, leakage-free feature pipeline and evaluated candidate models under grouped cross-validation by client domain to ensure cross-portfolio generalization. Our Histogram-based Gradient Boosting model achieved an observed **Precision@20 of 0.900** (18/20 correct decline identifications) on unseen client holdouts, outperforming the heuristic rule baseline (Precision@20 = 0.550) against a naive random base rate of 0.517. We translated model outputs into an operational Content Action Playbook featuring four distinct actionable archetypes, human review checklists, and automated drift triggers to provide reliable decision support for editorial triage.

---

## 1. Problem framing

### The Operational Decision
Digital publishing teams frequently manage portfolios spanning thousands of articles with constrained editorial bandwidth (typically 10–20 comprehensive rewrites per week). Editors need a reliable method to identify which specific pages are suffering active traffic decay and would benefit most from immediate refresh.

### Unit of Analysis & Outputs
- **Unit of Analysis**: One content item (article page) aggregated over a trailing 90-day activity window.
- **Output**: Calibrated decline probability score ($p \in [0, 1]$), rank ordering, reason code, and playbook action recommendation.
- **Cost of Errors**:
  - *False Positive*: An editor spends 3–4 hours updating a page whose traffic is naturally stable, wasting human bandwidth.
  - *False Negative (Costly)*: A high-visibility page in active decline is ignored, causing persistent ranking erosion from Page 1 to Page 3+ and substantial organic traffic loss.

Machine learning aids this decision by detecting subtle non-linear interactions across historical engagement, position drift, and staleness that simple single-threshold heuristic rules miss.

---

## 2. Data safety

### Dataset Specification
We utilized `data/raw/content_refresh_anonymized.csv`, comprising **30,000 rows × 44 columns** across **32 pseudonymized clients** representing 90 days of Google Search Console (GSC) and Google Analytics 4 (GA4) performance logs.

### Leakage Considerations & Exclusions
To ensure total methodological validity:
1. **Label-Derived Fields**: `trend_direction` and `trend_pct` were strictly excluded from features because the target label `is_declining_label` is defined directly as `(trend_direction == 'down')`.
2. **Sub-Window Comparison Features**: `impressions_last_30d`, `impressions_prev_30d`, `clicks_last_30d`, `clicks_prev_30d`, `sessions_last_30d`, and `sessions_prev_30d` were excluded. Including them allows models to compute the exact ratio defining the label, creating an artificial 0.999 ROC-AUC shortcut.
3. **Identifiers**: `content_id` and `client_id` were used exclusively for grouping and joining, never as model features.
4. **Privacy**: All client identifiers and URLs remain pseudonymized; no private client queries or raw URLs appear in the work.

---

## 3. Baseline

### Heuristic Rule Baseline (Week 4)
We formulated a transparent, rule-based priority score modeling standard industry practice:

$$\text{Baseline Score} = \text{is\_stale} \times \text{is\_visible} \times \log_2(1 + \text{impressions\_90d})$$

Where:
- $\text{is\_stale} = \mathbb{I}(\text{days\_since\_last\_update} \ge 180)$
- $\text{is\_visible} = \mathbb{I}(\text{impressions\_90d} \ge 100)$

On the held-out test split of 8 unseen clients (7,115 rows), this rule achieved:
- **Precision@20**: `0.550` (11/20 correct)
- **Precision@50**: `0.620` (31/50 correct)
- **Base Rate**: `0.517` (Random triage benchmark)

---

## 4. Model / analysis

### Model Architecture
We evaluated multiple model families (Logistic Regression, Decision Trees, Random Forests, and Gradient Boosting) using exclusively pre-decision observable signals:
- Trailing 90-day activity totals: `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `users_90d`, `engaged_sessions_90d`, `ai_sessions_90d`, `scroll_events_90d`, `days_with_impressions`, `days_with_sessions`.
- Content & metadata properties: `word_count`, `char_count`, `content_age_days`, `days_since_last_update`, `search_volume`, `competition`, `cpc`, `content_type`, `main_intent`.
- Derived rates: `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`, `ai_traffic_pct`.

Our selected model is a **Histogram-based Gradient Boosting Classifier** (`HistGradientBoostingClassifier`), configured with `max_depth=5`, `max_iter=200`, and `learning_rate=0.1`.

---

## 5. Evaluation

### Validation Design
To prevent data leakage and evaluate real-world transfer to new client domains, we implemented a **Grouped Shuffle Split** grouped by `client_id`:
- **Training Set**: 22,885 rows across 24 clients (75% client share).
- **Test Holdout**: 7,115 rows across 8 clients (25% client share, 0% domain overlap).

### Model vs. Baseline Results (Same Test Split)

| Model / Method | Precision@10 | Precision@20 | Precision@50 | ROC-AUC |
|---|---|---|---|---|
| **Naive Random Base Rate** | 0.517 | 0.517 | 0.517 | 0.500 |
| **Week-4 Rule Baseline** | 0.600 | 0.550 | 0.620 | n/a |
| **Logistic Regression** | 0.800 | 0.850 | 0.780 | 0.603 |
| **Decision Tree (depth=4)** | 0.600 | 0.600 | 0.560 | 0.598 |
| **Random Forest** | 0.400 | 0.450 | 0.620 | 0.602 |
| **HistGradientBoosting (Final)** | **0.900** | **0.900** | **0.860** | **0.620** |

The final model achieves a **+35 percentage point improvement in Precision@20** over the baseline rule on completely unseen client domains.

---

## 6. Interpretation

### Feature Importance & Signal Drivers
Permutation importance on the held-out test set identified the primary pre-decision drivers of decline:
1. `days_with_impressions`: Activity breadth is the strongest legitimate predictor; pages with intermittent daily impression presence are substantially more vulnerable to structural drop-offs.
2. `avg_position`: Higher average position (ranking slip past top 10) reflects active algorithmic depreciation.
3. `content_age_days` & `days_since_last_update`: Prolonged time without updates correlates monotonically with traffic decay.

### Error Analysis
- **False Positives (Top 50)**: False positives primarily involve niche reference articles with steady low-volume search demand that remained stable despite being un-updated.
- **Accuracy by Position Tier**: The model exhibits highest accuracy on `top_3` pages (79.1% accuracy) where signal-to-noise is cleanest, and lower accuracy on `deep` pages (>50 position, 50.4% accuracy) where impression counts are sparse and noisy.

---

## 7. Recommendation & Action Playbook

We mapped model scores and search properties into four actionable operational archetypes:

1. **`REFRESH_AND_EXPAND`** (`stale_high_volume_decay`): Pages un-updated $\ge 180$ days with $\ge 500$ 90-day impressions and high model risk. Allocated 3–4 hours for comprehensive editorial overhaul.
2. **`CTR_METADATA_OPTIMIZE`** (`strong_pos_weak_ctr`): Striking-distance pages (position $\le 20$) with below-benchmark CTR ($<0.5\%$). Allocated 0.5–1 hour for title tag and meta description optimization.
3. **`CONSOLIDATE_OR_RETIRE`** (`low_traffic_staleness`): Stale pages with minimal residual visibility ($<100$ impressions). Evaluated for 301 consolidation or archival.
4. **`MONITOR_MAINTAIN`** (`stable_healthy_asset`): Healthy, rising, or stable pages requiring zero current editorial intervention.

### Human Review Checklist & No-Go Boundaries
- **Must Check**: Search intent shifts, factual currency, keyword cannibalization, and technical crawlability.
- **Strictly Prohibited from Automation**: No automated 301 redirects, no direct-to-CMS auto-publishing without editorial sign-off, and no automated changes to YMYL or checkout pages.

---

## 8. Reproducibility

### Environment & Execution
- **Python**: 3.10+
- **Key Libraries**: `scikit-learn>=1.9.0`, `pandas>=3.0.0`, `numpy>=2.5.0`, `matplotlib>=3.11.0`
- **Random Seed**: `SEED = 42` fixed across all splits, models, and permutations.
- **Execution**:
  ```bash
  python -m venv .venv
  source .venv/bin/activate  # or .venv\Scripts\activate on Windows
  pip install -r requirements.txt
  # Run notebooks in work/notebooks/ in sequence: w01 -> w02 -> w03 -> w04 -> w05 -> w06 -> w07 -> capstone.ipynb
  ```

---

## 9. Acknowledgments & data credit

This research was conducted as part of the FlyRank Machine Learning Internship track.

*Built on the FlyRank ML Internship dataset* — [https://flyrank.ai](https://flyrank.ai).
