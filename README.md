# Mastercard Data Quest: Detecting Hidden Businesses in Transaction Data

ML solution by team **Outsiders** for Mastercard Data Quest (Mastercard × AIESEC, May 2026).

**Task:** find individuals who use personal cards for hidden commercial activity (taking payments for goods or services) instead of personal spending.

| Target | Meaning |
|---|---|
| `1` | hidden business |
| `0` | regular individual |

## Data

| Dataset | Description |
|---|---|
| `business_cards_MDQ.parquet` | transactions of known business cards (benchmark) |
| `consumer_cards_MDQ.parquet` | transactions of individual consumer cards |
| `merchants_reference.parquet` | merchant reference: MCC, country, merchant ID |

About 12.8M transactions and 105K cards, October 2025 to March 2026. Raw data is not included in this repository.

## Approach

1. **Data overview and quality checks.** Type conversion, duplicate and missing value checks, validated merge with the merchant reference (row counts are checked before and after).
2. **Exploratory analysis.** Business and consumer segments compared by average check, MCC structure, night activity and cross-border payments.
3. **Hypothesis testing.** 10 behavioral hypotheses were tested with chi-square tests. 7 of them showed significant differences and were turned into card-level features:
   1. Weekday night (02:00 to 06:00) non-lifestyle payments
   2. Advertising payments (MCC 7311)
   3. SaaS, cloud and digital subscriptions
   4. Household and advertising payments on the same card
   5. Household and IT/software payments on the same card
   6. Recurring payments in business MCC categories
   7. Online payments to foreign business merchants
4. **Reliable training set.** The full consumer portfolio may contain hidden businesses, so class `0` uses only "clean" consumers with no strong business signals. Uncertain consumers stay unlabeled until scoring.
5. **Model comparison.** Logistic Regression, Decision Tree, Random Forest, Hist Gradient Boosting and Extra Trees on a 60/40 split.
6. **Model selection.** Extra Trees was chosen: ROC-AUC ≈ 1.00 with zero false positives on validation, which matters most to avoid mislabeling regular customers. Hyperparameters were tuned and thresholds above 0.50 were reviewed.
7. **Scoring.** The final model scored about 19K uncertain consumer cards. Each card gets a probability and a potential level (Low, Medium or High) based on the probability and the number of strong business signals.

## Results

1. **11,540 cards (14.42%)** of the consumer portfolio were flagged as potential hidden businesses.
2. Final output: `final_consumer_hidden_business_predictions.csv` / `.parquet`, one row per consumer card with probability, potential level and binary prediction.
3. Business proposal: a strategy for Mastercard to help these clients move to business products, including an AI assistant in the banking app.

> The score is a behavioral signal for offering business products, not legal proof of business activity.

## Tech Stack

Python, pandas, NumPy, scikit-learn, SciPy, Matplotlib, Jupyter

## Repository Structure

```
├── MDQ_Outsiders.ipynb    # full pipeline: EDA, hypotheses, modeling, scoring
├── MDQ_Outsiders.pptx     # final presentation
└── README.md
```

## How to Run

1. Put the three `.parquet` files in the data folder set in `BaseDir`.
2. Install dependencies: `pip install pandas numpy scikit-learn scipy matplotlib pyarrow`
3. Run `MDQ_Outsiders.ipynb` from top to bottom.

## Team Outsiders

Alina Urazova, Alikhan Kemelkhan, Arina Rudkovskaya, Violeta Slepokurova

Our team also received a prize from Mastercard for the best post about the project.
