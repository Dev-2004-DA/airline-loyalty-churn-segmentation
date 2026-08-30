# Airline Loyalty Churn & Segmentation

Predicting silent churn and uncovering behavioral customer segments for a Canadian airline loyalty program — because most disengaged members never click "cancel."

## The Problem

Loyalty programs assume that if a member hasn't cancelled, they're still engaged. In reality, a large share of members quietly stop flying long before they ever cancel — and by the time they do cancel (if they ever do), the relationship is already over. This project set out to answer three questions for the airline's marketing team:

1. **Who is about to churn** — before it shows up as a formal cancellation?
2. **Who is actually valuable**, as opposed to who just *looks* valuable because of their card tier?
3. **What should the business do differently** for each type of customer?

## The Data

| Source | Rows | Description |
|---|---|---|
| Customer Loyalty History | 16,737 | One row per member — demographics, tier, salary, enrollment |
| Customer Flight Activity | 392,936 | One row per member per month, Jan 2017 – Dec 2018 |

Only **12.3%** of members had formally cancelled — far too low to use as a standalone churn definition, given how many members were flying zero flights a year while remaining "active" on paper.

## Approach

### 1. Data Understanding & Cleaning
- Missing salary (25% of rows) traced to college students with no income — filled as 0, not imputed.
- ~20 negative salary values corrected (typos) via absolute value.
- ~1,922 duplicate flight records identified and removed.
- Cancellation field confirmed unreliable as a sole churn signal.

### 2. Feature Engineering & Churn Labeling
Churn definition was engineered, not given: a member is **churned** if they formally cancelled **or** had zero flights across the full 24-month window (tested against 6-month and 12-month inactivity windows, both far too broad). This produced a churn rate of **16.0%** (2,686 / 16,737).

Key engineered features: `Months_Since_Last_Flight` (recency), `Tenure_Months`, `Redemption_Rate`, `Flight_Consistency_Score`, `Season_Concentration`, regional consolidation (11 provinces → 3 regions), plus one-hot and label encoding chosen per feature depending on whether categories were ordered.

Final modeling table: **16,737 rows × 33 columns**, zero nulls.

### 3. Modeling

**Task A — Churn Prediction (XGBoost)**
- 80/20 stratified split, class imbalance handled via `scale_pos_weight`, hyperparameters tuned with GridSearchCV (5-fold, recall-optimized).
- **Accuracy: 96% · Recall (churned): 95% · Precision (churned): 85% · AUC-ROC: 0.99**
- `Months_Since_Last_Flight` was the dominant predictor — roughly 6x more influential than any other feature.
- Customers scored into **High / Medium / Low risk tiers**; the High Risk tier (17.7% of members) had an 85.8% actual churn rate.

**Task B — Customer Segmentation (K-Means)**
- Churn label deliberately excluded from clustering features to avoid the segments simply mirroring the churn model.
- k=3 selected via elbow method + silhouette score (0.293, best among k=2–8).

## The Three Segments

| Segment | Size | Churn Rate | Profile |
|---|---|---|---|
| **Loyal Core Members** | 68.6% (11,482) | 2.0% | Long tenure (~46 mo), steady flyers, the stable backbone |
| **Disengaged At-Risk Members** | 26.4% (4,420) | 53.4% | Inactive ~21/24 months, redemption rate ~97% — cashing out before leaving |
| **High-Frequency New Flyers** | 5.0% (835) | 11.3% | New (~9 mo tenure), fly the most, but 0% redemption — never engaged with rewards |

**Key finding:** loyalty card tier (Star / Nova / Aurora) is almost identically distributed across all three segments — tier does not predict value or risk. Behavior does.

## Recommendations

| # | Recommendation | Target Segment | Owner |
|---|---|---|---|
| 1 | 90-day inactivity trigger in CRM for automated re-engagement | Disengaged At-Risk | CMO |
| 2 | "First Redemption" campaign before month 6 of membership | High-Frequency New Flyers | Marketing |
| 3 | Proactive, unprompted investment (perks, recognition) before risk appears | Loyal Core | CMO + CFO |

## Repository Structure

```
airline-loyalty-churn-segmentation/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   │   ├── Customer_Loyalty_History.csv
│   │   └── Customer_Flight_Activity.csv
│   └── processed/
│       ├── modelling2.csv
│       └── final_customer_segments.csv
│
├── notebooks/
│   └── Airline_Loyalty_Complete_Report.ipynb
│
├── app/
│   └── app.py
│
├── reports/
│   ├── Airline_Loyalty_NonTechnical_final_Report.pdf
│   └── Airline_Loyalty_Project_Detail_Report.docx
│
├── images/
│
└── docs/
    └── dashboard_link.md
```

## Tools & Stack

Python · pandas · scikit-learn · XGBoost · Streamlit · Jupyter

## Live Dashboard

[Streamlit App](https://airlineloyaltyretention-fcazb9b8zccwwk6gtzudaf.streamlit.app/) — Executive Summary, Targeted Campaign Planner, and Customer Lookup tabs.

## Author

Consulting & Analytics Club, IIT Guwahati — Summer Projects 2026
