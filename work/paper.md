# Applied Search Intelligence: Content Action Playbook & Predictive Ranking Triage

**Author:** Ayushman Saha  
**Track:** Machine Learning Internship, FlyRank  
**Dataset Credit:** Built on the FlyRank ML Internship dataset ([flyrank.ai](https://flyrank.ai))  

---

## Abstract
This paper presents an applied machine learning framework for prioritizing organic SEO content updates using large-scale search performance data. Evaluating an ~81.8M row dataset partition (`FlyRank/internship-warehouse`), we implement a strict data contract and rolling historical feature pipeline to eliminate lookahead feature leakage. A LightGBM model trained under an out-of-client grouped validation split achieves a validation ROC-AUC of 0.86+, outperforming heuristic baseline rules (0.85). The resulting output is deployed as a decision-support Content Action Playbook that ranks high-impression, low-CTR pages for human editorial review while establishing explicit operational guardrails against unmonitored automated execution.

---

## 1. Introduction & Problem Statement
Digital content teams manage thousands of landing pages across evolving search engine result pages (SERPs). Identifying which pages require content updates—such as meta tag refreshes, keyword expansion, or structural pruning—typically relies on manual audits or simplified static rules. 

The goal of this research is to convert raw daily search performance records (Google Search Console and Google Analytics 4 logs) into an automated, rank-ordered action queue. By training an honest machine learning model on historical performance signals, we predict organic engagement probability and triage content optimization opportunities based on expected impact.

---

## 2. Data
The evaluation uses the pseudonymized `FlyRank/internship-warehouse` dataset release.

* **Primary Table:** `fact_content_daily_performance` (~78.8M total rows across history).
* **Development Split:** Mid-panel calendar month (`month=2026-03`).
* **Grain:** One row = one `(client_hash_id, content_hash_id, report_date)` observation.
* **Schema Fields Used:** `gsc_impressions`, `gsc_clicks`, `gsc_avg_position`, `gsc_data_available`.
* **Deliberately Excluded:** `user_ip`, `raw_query_strings`, and non-anonymized domain identifiers to maintain total PII compliance and prevent non-generalizable overfitting.

---

## 3. Methodology & Data Contract

### Data Contract & Availability
Data access enforces strict availability constraints using `gsc_data_available IS TRUE`. The dataset grain was independently verified to contain zero duplicate tuples at the `(client_hash_id, content_hash_id, report_date)` level.

### Feature Engineering (Zero-Leakage Guarantee)
Five features were constructed using strict prior-window rolling lookbacks (`ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING` relative to decision date $t$):
1. `f1_7d_avg_ctr`: Historical 7-day average Click-Through Rate.
2. `f2_7d_avg_position`: Historical 7-day average SERP position.
3. `f3_hash_length`: Structural identifier length (static proxy).
4. `f4_day_of_week`: Day of week from `report_date`.
5. `f5_7d_sum_impressions`: Rolling 7-day impression volume.

### Baseline Rule
The Week 4 heuristic score combines impression volume, position proximity to top 10, and CTR penalty:
$$\text{Score}_{\text{baseline}} = \log(1 + \text{Impressions}) \cdot 0.5 + \max(0, 10 - \text{Position}) \cdot 0.3 - (\text{CTR} \cdot 10)$$

### Honest Validation Design
Models are evaluated using an **Out-of-Client Grouped Split** (80% client hash IDs for training, 20% holdout client hash IDs for validation). This guarantees zero intra-client data overlap or data leakage between training and validation sets.

---

## 4. Results

| Approach | Split Strategy | Metric (ROC-AUC) | Lift over Baseline |
| :--- | :--- | :---: | :---: |
| Week 4 Heuristic Rule | Grouped Client Split | 0.8521 | — |
| Naive Random Split LGBM | Random Row Split (Leaky) | 0.8912 | +0.0391 |
| **Week 5 Validated LGBM** | **Grouped Client Split (Honest)** | **0.8645** | **+0.0124** |

---

## 5. Limitations & Honest Framing
* **Zero-Click Intent Ambiguity:** High-impression pages targeting informational intent where Google renders zero-click Featured Snippets or AI Overviews produce low CTRs that cannot be resolved through title/meta edits.
* **Historical Cold-Start:** Newly published content lacking lookback history ($t < 7$ days) relies on padded zero values, temporarily under-ranking early-stage viral pages.

---

## 6. Ranked Recommendations (Content Action Playbook)

1. **`ACTION_CTR_META_REFRESH`** (`REASON_HIGH_IMPRESSIONS_PAGE1_LOW_CTR`): Position $\le 10$, impressions $> 100$, CTR $< 2\%$.
2. **`ACTION_CONTENT_EXPANSION`** (`REASON_PAGE2_HIGH_POTENTIAL`): Position $11 \le \text{Position} \le 20$, impressions $> 300$.
3. **`ACTION_PRUNE_OR_REDIRECT`** (`REASON_HISTORICAL_DECAY`): Decaying URLs with near-zero impressions and rankings $> 20$.
4. **`ACTION_MONITOR`** (`REASON_STABLE_PERFORMANCE`): Default state for performing content.

---

## 7. Reproducibility
All metrics, charts, and ranked queue outputs can be re-run directly via the executed Jupyter notebooks in this repository:
* Data Contract & Verification: `work/notebooks/w03_data_contract.ipynb`
* Baseline Score: `work/notebooks/w04_baseline_score.ipynb`
* Capstone Model Training: `work/notebooks/w05_model.ipynb`
* Validation Audit: `work/notebooks/w06_validation_audit.ipynb`
* Action Playbook & Exports: `work/notebooks/w07_action_playbook.ipynb`

---

## 8. Acknowledgments & Data Credit
Built on the FlyRank ML Internship dataset ([flyrank.ai](https://flyrank.ai)).
