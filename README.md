# Cafe Sales — Data Cleaning (Task 1)

A professional-style data cleaning project: a deliberately messy café sales dataset (10,000 transactions) is profiled, cleaned, validated and saved as an analysis-ready CSV. Every decision is documented in the notebook.

## Files

| File | Description |
|---|---|
| `cafe_sales_cleaning.ipynb` | The full cleaning workflow (executed, with outputs and markdown justifications) |
| `dirty_cafe_sales.csv` | Raw input data (10,000 rows × 8 columns) |
| `clean_cafe_sales.csv` | Cleaned output (9,977 rows × 8 columns) |

## Tech stack

Python 3 · pandas · numpy · Jupyter Notebook

## How to run

```bash
pip install pandas numpy jupyter
jupyter notebook cafe_sales_cleaning.ipynb
```

Keep `dirty_cafe_sales.csv` in the same folder as the notebook, then run **Kernel → Restart & Run All**. The cleaned file is written to `clean_cafe_sales.csv`.

## The dataset

| Column | Meaning |
|---|---|
| Transaction ID | Unique transaction identifier (`TXN_xxxxxxx`) |
| Item | Menu item sold (Coffee, Tea, Cake, Cookie, Juice, Salad, Sandwich, Smoothie) |
| Quantity | Units purchased (1–5) |
| Price Per Unit | Menu price of the item |
| Total Spent | Quantity × Price Per Unit |
| Payment Method | Cash, Credit Card or Digital Wallet |
| Location | In-store or Takeaway |
| Transaction Date | Date of sale (2023) |

## Problems found in the raw data

- **Missing values hidden as text:** empty cells, `UNKNOWN` and `ERROR` — there are no true `NaN`s, so a naive `isnull()` reports nothing.
- **High missingness** in Payment Method (~32%) and Location (~40%).
- **Every column loaded as text**, so numbers and dates could not be used.
- No duplicate rows or duplicate Transaction IDs.

## Cleaning approach

1. **Standardise** — strip whitespace, unify casing, convert all placeholder tokens to real nulls, parse numbers and dates.
2. **Recover before imputing** — each item has one fixed price and `Total = Quantity × Price`, so missing Item, Quantity, Price and Total values were reconstructed exactly (about 2,000 values) instead of being estimated.
3. **Handle remaining gaps per column:**

   | Column | Strategy |
   |---|---|
   | Price / Quantity / Total | Recovered from business rules |
   | Item | Recovered when the price identifies one item; otherwise `Unknown` |
   | Payment Method, Location | Labelled `Unknown` (avoids fabricating transactions via mode imputation) |
   | Transaction Date | Left as `NaT` (rows are unordered, so forward-fill would be wrong) |
   | Rows with unrecoverable Total | Deleted (23 rows) |

4. **Duplicates** — checked on the full row and on Transaction ID; none found, none removed.
5. **Data types** — IDs to `string`, labels to `category`, Quantity to `Int64`, money to `float64`, dates to `datetime`.
6. **Outliers** — IQR and Z-score applied to all numeric columns. The 269 `Total Spent` values flagged by IQR are valid menu combinations (maximum 5 × 5.00 = 25), so they were **retained**.
7. **Validation** — `Total = Quantity × Price` holds for every row where both are known.

## Results: before vs. after

| Metric | Before | After |
|---|---|---|
| Row count | 10,000 | 9,977 |
| Null cells (incl. `UNKNOWN`/`ERROR`/empty) | 10,082 | 466 |
| Duplicate rows | 0 | 0 |
| Dtype accuracy | 4/8 (50%) | 8/8 (100%) |

**Remaining nulls (466):** 460 missing Transaction Dates and 3 rows each with Quantity and Price blank (only the total is known; the revenue figure is valid, so the rows are kept). Item, Payment Method and Location use an explicit `Unknown` label, which is a category, not a null.

## Notes and limitations

- Missing dates (4.6%) mean those rows drop out of any time-series analysis.
- `Unknown` is a sizeable share of Payment Method and Location, so analyses of those fields should account for it.
- Item cannot be recovered when the price is shared by two items (3.00: Cake/Juice, 4.00: Sandwich/Smoothie).


# Retail Sales — Exploratory Data Analysis (Task 2)

An exploratory analysis of 1,000 retail transactions covering sales trends, customer demographics, product performance and the key drivers of revenue. The notebook ends with business recommendations, and a statistical check separates real patterns from chance.


## Tech stack

Python 3 · pandas · numpy · matplotlib · seaborn · scipy · Jupyter Notebook

## How to run

```bash
pip install pandas numpy matplotlib seaborn scipy jupyter
jupyter notebook retail_sales_eda.ipynb
```

Keep `retail_sales_dataset.csv` in the same folder as the notebook, then choose **Kernel → Restart & Run All**. Charts are also saved to `figures/`.

## The dataset

| Column | Meaning |
|---|---|
| Transaction ID | Unique ID per transaction |
| Date | Purchase date (2023-01-01 to 2024-01-01) |
| Customer ID | Unique per customer (each customer appears once) |
| Gender | Female / Male |
| Age | 18–64 |
| Product Category | Beauty, Clothing, Electronics |
| Quantity | 1–4 units |
| Price per Unit | 25, 30, 50, 300 or 500 |
| Total Amount | Quantity × Price per Unit |

The data is already clean: no nulls, no duplicates, and `Total Amount` is consistent in every row.

## What the notebook covers

| Checklist item | Where |
|---|---|
| Shape, dtypes, null check | Section 1 |
| Mean, median, mode, std for numeric columns | Section 2 |
| Monthly and quarterly sales line charts | Section 3 |
| Age group and gender breakdown | Section 4 |
| Top 10 best-sellers; revenue by category | Section 5 |
| Correlation heatmap | Section 6 |
| Additional visualisations (premium share, weekday, age × category) | Section 7 |
| Statistical significance checks | Section 8 |
| Conclusions and recommendations | Section 9 |

## Key findings

- **Price level drives revenue, not who the customer is.** Premium items (priced 300–500) are **40% of orders but 88% of revenue**. A premium order averages about 12× a budget order. Total Amount correlates strongly with Price per Unit (r = 0.85) and moderately with Quantity (r = 0.37).
- **Sales are uneven through the year.** Peaks in **May (53,150)** and **October (46,580)**, a deep slump in **September (23,620)**. Q3 is about 22% below Q2, and Q4 is the strongest quarter (126,190).
- **Demographics are balanced.** The gender split is 51% female / 49% male, with ages spread evenly across 18–64.
- **Categories are balanced.** Electronics 34.4%, Clothing 34.1%, Beauty 31.5% of revenue; no single product line dominates unit sales.
- **Most apparent demographic differences are not statistically significant.** Gender, age group and weekday differences all fail the significance test (p > 0.05); the age trend in order value is borderline (p ≈ 0.056). Only the premium-vs-budget gap is clearly real.

## Recommendations (summary)

1. Make premium selling the core commercial strategy (upsell prompts, premium-first placement, bundles).
2. Plan the calendar around the May/October peaks and run a targeted campaign to lift the September slump.
3. Use broad, category- and price-led targeting rather than gender or age segmentation; A/B test the borderline 18–25 signal.
4. Start capturing product names/SKUs, a persistent customer ID, sales channel and discounts to enable deeper analysis.

Details and the reasoning behind each are in Section 9 of the notebook.

## Assumptions and limitations

- **No product names exist in the data.** For the "top 10 best-selling products" a product is defined as **category + unit price** (e.g. "Electronics @ 500"), giving 15 product lines.
- **No currency is stated**, so amounts are in plain currency units.
- Only **2 transactions fall in 2024**, so monthly and quarterly charts use 2023 only.
- Each customer appears **once**, so repeat purchase and loyalty cannot be analysed.
- One year of data means seasonality is suggestive, not proven. There is no cost or margin data, so revenue is analysed, not profit.


# Credit Card Fraud Detection (Task 3)

Exploratory analysis and data preparation for detecting fraudulent credit card transactions. The data is **extremely imbalanced** (fraud is only about 0.17% of transactions), so the project focuses on understanding that imbalance, why plain accuracy is a misleading metric, and how to prepare the data for a fair model.

> **Status:** work in progress. Exploration, the train/test split and SMOTE resampling are done. Model training and evaluation are the next step (see [Next steps](#next-steps)).



The dataset itself is not included because of its size. See [Dataset](#dataset) below.

## Tech stack

Python 3 · pandas · matplotlib · seaborn · scikit-learn · imbalanced-learn · Jupyter Notebook

## Dataset

`creditcard.csv`: **284,807 transactions × 31 columns**, with no missing values.

| Column | Description |
|---|---|
| `Time` | Seconds elapsed since the first transaction in the dataset |
| `V1` – `V28` | Anonymised numerical features (the output of a PCA transformation) |
| `Amount` | Transaction amount |
| `Class` | Target: `0` = legitimate, `1` = fraud |

This appears to be the widely used **Credit Card Fraud Detection** dataset on Kaggle (ULB Machine Learning Group, `mlg-ulb/creditcardfraud`). Download `creditcard.csv` from there and place it next to the notebook.

## What the notebook does

1. **Load and inspect** the data: shape, columns, dtypes and null check (all 31 columns complete, 30 float and 1 integer).
2. **Measure the class imbalance:**

   | | Count |
   |---|---|
   | Total transactions | 284,807 |
   | Fraud transactions | 492 |
   | Fraud percentage | **0.173%** |

3. **Fraud vs. non-fraud amounts:** box plot of `Amount` by `Class`.
4. **Time-of-day analysis:** creates an `Hour` feature from `Time` and plots legitimate vs. fraud transactions by hour.
5. **Why accuracy is misleading:** a model that predicts "not fraud" for everything would score about 99.8% accuracy while catching **zero** fraud. The notebook therefore uses **Precision, Recall, F1-score, the confusion matrix and ROC-AUC**, with **Recall** as the key metric (it measures how many real frauds are caught).
6. **Stratified train/test split** (80/20, `random_state=42`), which keeps the fraud ratio the same in both sets:

   | | Legitimate | Fraud |
   |---|---|---|
   | Training set | 227,451 | 394 |
   | Test set | 56,864 | 98 |

7. **SMOTE oversampling on the training data only**, which balances the training set to 227,451 legitimate and 227,451 fraud samples. The test set is left untouched so that evaluation reflects real-world, imbalanced data.

## How to run

```bash
pip install pandas matplotlib seaborn scikit-learn imbalanced-learn jupyter
jupyter notebook Creditcard.ipynb
```

Before running, update the data path in the loading cell to where `creditcard.csv` is on your machine, for example:

```python
card = pd.read_csv("creditcard.csv")
```

## Notes

- **`Hour` is relative, not clock time.** `Time` counts seconds from the first transaction, so `Hour` shows position within each 24-hour cycle of the data, not the true time of day.
- **Resample only the training data.** Applying SMOTE before the split would leak synthetic copies of test information into training and inflate results. The notebook correctly resamples after splitting.

