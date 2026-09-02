# Historical Signals for Identifying Declining Content

## Abstract
Can historical search, traffic, engagement, and content-freshness signals identify content that is currently showing a declining trend? Using 30,000 anonymized content observations, I defined decline from the observed `trend_direction` label and excluded label-derived fields from the model features. I compared a transparent majority-class baseline with logistic regression under client-grouped validation, using 23,837 training observations and 6,163 test observations with zero client overlap. The grouped test result reached **0.8497 ROC-AUC, 75.61% accuracy, 80.71% precision, 68.69% recall, and 74.22% F1**, compared with a 0.50 ROC-AUC majority baseline. The result is best treated as decision-support for prioritizing human content review rather than as a causal or autonomous publishing decision.

## 1. Research Question
**Can historical search, traffic, engagement, and content-freshness signals identify content that is currently showing a declining trend, and which signals provide the most useful evidence for prioritizing content review?**

## 2. Problem & Decision Context
A large content inventory cannot be reviewed with equal depth at every moment. The practical goal is therefore not to automate content decisions, but to identify observations that deserve earlier human attention. The unit of analysis is a content/page observation. The model output is a binary classification signal indicating whether the observed trend is labeled as declining.

## 3. Data
The working dataset contains **30,000 observations and 44 columns**. The target was reconstructed as `is_declining_label = 1` when `trend_direction == "down"`, otherwise `0`. Across the full working dataset, the declining-label rate is **54.21%**.

The model uses 28 non-label-derived features covering search demand and competition, content characteristics, historical and recent traffic, engagement, and content freshness.

### Deliberate exclusions
`trend_direction` was excluded because it directly defines the target. `trend_pct` was also excluded because it directly describes the trend used to create the target. These fields would create target leakage rather than independent predictive evidence.

Client and content identifiers are used only for grouping or reproducibility where needed; they are not presented as public analytical features.

### Data safety
The public-facing report does not expose client names, client-identifying information, private queries, or raw content identifiers. The analysis uses anonymized working data and reports aggregate findings.

## 4. Methodology

### 4.1 Target definition
- `1` = labeled declining (`trend_direction == "down"`)
- `0` = otherwise

This is a classification framing, not a claim that the model discovers an underlying causal state.

### 4.2 Leakage checks
The analysis explicitly removes fields derived from the target. Recent performance variables remain in the feature set because they represent historical signals available to the analysis, but their proximity to the outcome means their coefficients should be interpreted conservatively.

### 4.3 Baseline
The transparent baseline predicts the majority class for every observation in the grouped test set. The grouped test set contains **3,149 declining and 3,014 non-declining observations**, so the baseline accuracy is **51.09%** and its ROC-AUC is **0.50**.

### 4.4 Model
The primary model is **Logistic Regression** with median imputation and feature standardization. Logistic regression was selected because its behavior is relatively transparent and its coefficients provide directional evidence about standardized features and the classification label.

### 4.5 Validation design
The final evaluation uses **GroupShuffleSplit**, grouping by `client_id`, with random seed 42 and a 20% test allocation.

- Training observations: **23,837**
- Test observations: **6,163**
- Training clients: **25**
- Test clients: **7**
- Client overlap: **0**

A prior random split produced stronger metrics, but it allows observations from the same client to appear on both sides of the split. The grouped split is therefore the main result because it provides a more conservative estimate of generalization across clients.

## 5. Results

### 5.1 Headline result

| Metric | Logistic Regression | Majority Baseline |
|---|---:|---:|
| ROC-AUC | **0.8497** | 0.5000 |
| Accuracy | **75.61%** | 51.09% |
| Precision | **80.71%** | 51.09% |
| Recall | **68.69%** | 100.00% |
| F1 | **74.22%** | 67.63% |

The model provides substantial discrimination beyond the majority-class baseline on the same grouped test set.

### 5.2 Random vs grouped validation
The earlier random split produced **0.9124 ROC-AUC** and **83.39% F1**. Under client-grouped validation, those values fell to **0.8497 ROC-AUC** and **74.22% F1**. This gap indicates that random splitting likely gave an optimistic estimate of generalization compared with holding entire clients out of the test set.

### 5.3 Confusion matrix

| | Predicted non-declining | Predicted declining |
|---|---:|---:|
| **Actual non-declining** | 2,497 | 517 |
| **Actual declining** | 986 | 2,163 |

The model produced **2,163 true positives**, **2,497 true negatives**, **517 false positives**, and **986 false negatives**. The larger false-negative count is operationally important, so the model should be treated as prioritization support rather than a complete detection system.

## 6. Interpretation
The strongest practical signals are best understood as evidence for review rather than explanations of why a page declined. The available feature groups cover search visibility, recent traffic, engagement, and freshness.

The action-playbook analysis identifies useful human-readable reason codes such as:

1. **Low search position** — inspect search visibility and content/query alignment.
2. **Recent impression decline** — investigate whether search exposure has weakened.
3. **Recent session decline** — inspect changes in downstream traffic.
4. **Content staleness** — check whether the page needs a substantive refresh.
5. **No recent impressions** — verify whether the page still receives meaningful search exposure.

These signals should be reviewed together rather than interpreted independently.

## 7. Limitations & Honest Framing

### Observational data
This analysis does not establish that any feature causes decline. It measures associations with an observed label.

### Recent-performance proximity
Some recent traffic and search variables are close in time to the outcome. Their predictive usefulness may therefore partly reflect information near the labeling window rather than long-horizon forecasting ability.

### Client distribution
The grouped test contains seven held-out clients. This is stronger than allowing client overlap, but it is still a limited sample of independent client groups.

### False negatives
The model missed 986 declining observations in the grouped test set. It should not be treated as a complete monitoring or alerting system.

### No autonomous action
The results do not justify automatic deletion, publishing, rewriting, or other irreversible content decisions. They support human review prioritization.

## 8. Ranked Recommendations

### Priority 1 — Use the model as a review-prioritization signal
Route high-confidence model outputs into a human review queue. Require the reviewer to inspect the underlying search, traffic, engagement, and freshness evidence before taking action.

### Priority 2 — Combine model evidence with reason codes
Do not show a score without context. Pair a model signal with human-readable evidence such as search-position weakness, recent impression decline, session decline, or stale content.

### Priority 3 — Validate on future client groups
Before operational adoption, repeat evaluation on a fresh holdout containing clients and time periods that were not used during model development. Track precision, recall, ROC-AUC, and false-negative rates over time.

### Action playbook

| Signal | Human review | Suggested next check |
|---|---|---|
| Low search position | High priority | Search visibility and content alignment |
| Impression decline | High priority | Compare recent vs previous search exposure |
| Session decline | Medium/high priority | Inspect traffic changes and engagement |
| Stale content | Medium priority | Check factual freshness and update need |
| No recent impressions | Verification | Confirm current search exposure before editing |

## 9. Reproducibility
The analysis is organized in the repository notebooks under `work/notebooks/`.

Key artifacts include:
- `w03_feature_leakage_check.ipynb` — leakage review
- `w04_baseline_score.ipynb` — baseline/action-score work
- `w05_model.ipynb` — model development
- `w06_validation_audit.ipynb` — grouped validation and final evaluation
- `w07_action_playbook.ipynb` — ranked review actions
- `capstone.ipynb` — capstone notebook skeleton

The final validation uses a fixed random seed of **42** and client-grouped splitting. The report should be regenerated from the notebooks after any change to preprocessing, features, split design, or model configuration.

## 10. Conclusion
Historical search, traffic, engagement, and freshness signals contain measurable information associated with the observed declining label. Under the more conservative client-grouped validation, logistic regression reached **0.8497 ROC-AUC** and **74.22% F1**, clearly outperforming the majority-class baseline on the same test set. The reduction from random-split performance demonstrates why grouped validation matters when multiple observations belong to the same client. The practical value is therefore not an autonomous content decision engine, but a transparent prioritization layer that helps humans decide what to investigate first.

## 11. Acknowledgments & Data Credit
**Built on the FlyRank ML Internship dataset** — [FlyRank](https://flyrank.ai).

This project was completed as part of the FlyRank ML Internship capstone work. All public-facing claims are intentionally framed as measured or directional findings from the available anonymized working dataset.
