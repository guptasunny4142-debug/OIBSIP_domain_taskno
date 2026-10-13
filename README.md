# Café Sales — Data Cleaning (Task 3)

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
