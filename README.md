# Final project for MSBA 510 (Python for Business Analytics) at CSUCI (Working Draft)
Ingests UK online retail transactions (2009–2011), cleans them through bronze, silver, and gold layers in DuckDB, and models customer lifetime value and churn risk in Jupyter notebooks. Customer segments feed campaign recommendations. 

Data source: UCI Machine Learning Repository, Online Retail II.
Dataset link: https://archive.ics.uci.edu/dataset/502/online+retail+ii

## Project structure

```
online-retail-ltv-churn-medallion/
├── notebooks/
│   ├── 01_ingest_bronze.ipynb
│   ├── 02_silver_clean.ipynb
│   ├── 03_gold_rfm_cohorts.ipynb
│   ├── 04_ltv_churn_model.ipynb
│   └── 05_segments_recs.ipynb
├── data/                      (created on first run, not stored in GitHub)
│   ├── raw/                   downloaded dataset (.xlsx and .csv)
│   ├── ltv.duckdb             database with the bronze and silver tables
│   └── exports/               CSV outputs from notebooks 03 to 05
├── requirements.txt
├── .gitignore
└── README.md
```

The `data/` folder is listed in `.gitignore`, so the dataset and database are never uploaded. Anyone who clones the repo rebuilds them by running the notebooks. All paths in the notebooks are relative to the project folder, so the project runs on any computer without editing file paths.

## Output files

All outputs are saved to `data/exports/`.

**`gold_customers.csv`** (notebook 03, one row per customer)

| Column | Meaning |
|---|---|
| `Customer_ID` | Customer identifier |
| `first_purchase`, `last_purchase` | Dates of the first and last order |
| `frequency` | Number of separate orders (invoices) |
| `monetary` | Total spend |
| `recency_days` | Days since the last purchase |
| `tenure_days` | Days since the first purchase |
| `first_purchase_month` | Month of the first purchase (the customer's cohort) |

**`customer_scores.csv`** (notebook 04, one row per customer)

| Column | Meaning |
|---|---|
| `repeat_buyer` | True if the customer bought on more than one day |
| `frequency` | Number of *repeat* purchase days (the first purchase is not counted, so this differs from the gold table) |
| `recency` | Days between the first and last purchase |
| `T` | Days between the first purchase and the end of the data |
| `monetary_value` | Average value of repeat orders, capped at the 95th percentile |
| `p_alive` | Probability the customer is still active |
| `churn_risk` | 1 minus `p_alive`. Only meaningful for repeat buyers. |
| `pred_purchases_12m` | Expected purchases in the next 12 months |
| `pred_order_value` | Expected value of each order |
| `ltv_12m` | Predicted 12-month revenue from the customer |
| `ltv_decile` | 1 to 10, where 10 is the top 10% by predicted LTV |

**`customer_segments.csv`** (notebook 05): `customer_scores.csv` plus a `segment` column.

**`segment_recommendations.csv`** (notebook 05): one row per segment with the recommended campaign, spend level, customer count, and estimated extra revenue.

## Business questions

The project answers five questions a marketing team would ask about its customer base:

1. **What do our customers look like?** How recently they bought, how often, and how much they spend.
2. **Where does the revenue come from?** How concentrated it is in the best customers, and when in the year it arrives.
3. **Do customers come back?** How many new customers become repeat buyers.
4. **Who is worth the most, and who is leaving?** Predicted 12-month value and churn risk for every customer.
5. **What should we do about it?** Customer segments, with a recommended campaign for each.

## Tools

| Package | Used for |
|---|---|
| `duckdb` | Local database for the bronze and silver tables |
| `pandas` | Data cleaning and analysis |
| `openpyxl` | Reading the downloaded Excel file |
| `matplotlib`, `seaborn` | Charts |
| `lifetimes` | BG/NBD and Gamma-Gamma models |
| `scikit-learn` | Logistic regression and model scoring |
| `jinja2` | Formatting tables with $ and % in the notebooks |

## Setup

**Requirements:** Python 3.10 or newer and an internet connection (the first run downloads the dataset, about 45 MB).

1. **Clone the repo and open it in VS Code or Jupyter**
   ```
   git clone https://github.com/Jshort805/online-retail-ltv-churn-medallion.git
   cd online-retail-ltv-churn-medallion
   ```

2. **(Recommended) Create a virtual environment**
   ```
   python -m venv .venv
   source .venv/bin/activate        #Mac/Linux
   .venv\Scripts\activate           #Windows
   ```

3. **Install the packages**
   ```
   pip install -r requirements.txt
   ```

4. **Pick the Python kernel** you just installed into (in VS Code, click "Select Kernel" at the top right of a notebook).

## How to run

Run the notebooks **in order**, using Run All in each one. Each notebook reads what the previous one saved.

| Notebook | What it does | Saves |
|---|---|---|
| `01_ingest_bronze` | Downloads the UCI dataset and loads it into DuckDB | `bronze` table |
| `02_silver_clean` | Cleans the data and flags cancellations, returns, and missing customer IDs. It asks "yes/no" before saving, so type `yes` | `silver` table |
| `03_gold_rfm_cohorts` | Builds one row per customer (recency, frequency, spend) and explores the data | `data/exports/gold_customers.csv` |
| `04_ltv_churn_model` | Predicts customer lifetime value and churn risk, and tests the predictions on held-out data | `data/exports/customer_scores.csv` |
| `05_segments_recs` | Builds customer segments and campaign recommendations | `data/exports/customer_segments.csv`, `data/exports/segment_recommendations.csv` |

**Troubleshooting**
- **DuckDB "database is locked" error:** only one notebook can use `data/ltv.duckdb` at a time. Close the other notebooks, or restart the kernel, and run again.
- **Notebook 02 seems stuck:** it stops and waits for you to type `yes` or `no` before saving. Type `yes` in the box at the top of the window and press Enter.
- **"No module named ..." error:** the notebook is using a different Python than the one you installed into. Pick the `.venv` kernel (Setup step 4) and run `pip install -r requirements.txt` again.
- **Notebook 04 or 05 can't find a file or table:** an earlier notebook hasn't been run yet. Run them in order, 01 to 05.

## How the data flows

The project follows a **medallion** layout: each layer is a cleaner version of the one before it.

| Layer | Where it lives | What it holds |
|---|---|---|
| **Bronze** | `bronze` table in DuckDB | The raw data exactly as downloaded: 1,067,371 invoice lines, 8 columns |
| **Silver** | `silver` table in DuckDB | The same rows, cleaned, with flag columns added. No rows are deleted. |
| **Gold** | `data/exports/gold_customers.csv` | One row per customer (5,878 customers) |
| **Model outputs** | `data/exports/` | Customer scores, segments, and recommendations |

**What the cleaning does (notebook 02).** Text is trimmed and lowercased, and column names are tidied. Instead of deleting messy rows, the notebook adds flags so later notebooks can choose what to filter:

| Flag | Meaning | Rows flagged |
|---|---|---|
| `Customer_ID_Exists` | False when the row has no customer ID | 243,007 without an ID |
| `Cancelled_Order` | Invoice number starts with "c" | 19,494 |
| `Return_Adjustment` | Negative quantity or price on a non-cancelled invoice | 3,462 |
| `Missing_Data` | Any value in the row is missing | 243,007 |

**What counts as a real sale (notebooks 03 and 04).** A row is kept only if it has a customer ID, is not a cancellation or return/adjustment, and has a quantity and price above zero. That leaves 805,549 rows.

## Methods

- **RFM (notebook 03).** Each customer is described by recency (days since last purchase), frequency (number of orders), and monetary (total spend), measured as of the day after the last invoice in the data.
- **Cohort retention (notebook 03).** Customers are grouped by the month of their first purchase, and we track what share of each group buys again in each later month.
- **BG/NBD (notebook 04).** Predicts how many purchases a customer will make and the probability they are still active (`p_alive`). It suits stores like this one, where customers buy whenever they want and never formally cancel.
- **Gamma-Gamma (notebook 04).** Predicts the value of each future order. Average order values are capped at the 95th percentile (about $997) because a few very large wholesale orders break the model.
- **Lifetime value.** Predicted 12-month LTV = expected purchases in 12 months x expected order value.
- **Honest testing.** The model is fit only on purchases before June 9, 2011, then checked against what customers actually did in the following 183 days. The split is by date, not random, so the model never sees the future. It is also compared against a simple baseline.
- **Churn comparison.** A customer churned if they bought before June 9, 2011 and bought nothing in the test window. A logistic regression is compared with BG/NBD on repeat buyers, using AUC.
- **Segments (notebook 05).** Repeat buyers are split on value (predicted LTV at or above the repeat-buyer median, $797) and risk (churn risk of 0.25 or more). One-time buyers are their own segment.

## Key findings

All numbers come from running the notebooks on the full dataset (1,067,371 rows, December 2009 to December 2011).

**The customer base (notebook 03)**
- 5,878 customers placed real orders. 27.6% ordered only once, and 85% ordered 10 times or fewer.
- Revenue is concentrated: the top 1% of customers bring in 31.8% of revenue, the top 20% bring in 77.2%, and the bottom half only 6.4%.
- The business is seasonal. Revenue peaks every November at about double a normal month.
- Most new customers do not become regulars. In a typical later month, only 10 to 20% of a cohort buys again.

**Lifetime value and churn (notebook 04)**
- The purchase model (BG/NBD) was trained only on purchases before June 9, 2011, then tested on the following 6 months, which it never saw. It missed by about 1 purchase per customer on average (MAE 1.047) and beat a simple baseline that assumes each customer keeps buying at their past pace (MAE 1.095).
- Predicted revenue for the next 12 months is about $7.6 million, and the top 10% of customers hold 52% of it.
- 74% of one-time buyers did not buy again in the 6-month test window, compared with 36% of repeat buyers.
- BG/NBD and a logistic regression rank churn risk equally well (AUC 0.782 and 0.781). We use BG/NBD because the same model also gives expected purchases and lifetime value.

**Segments and recommendations (notebook 05)**

| Segment | Customers | Share of predicted revenue | Recommended campaign |
|---|---|---|---|
| Champions (high value, low risk) | 2,063 | 82% | Loyalty perks and early access. No discounts. |
| At-risk high value | 32 | 1% | Personal win-back offer and outreach |
| Loyal low-value | 1,431 | 8% | Cross-sell and bundles to raise order size |
| Fading low-value | 663 | 2% | Automated reactivation email only |
| One-time buyers | 1,689 | 7% | Second-purchase offer soon after the first order |

Under the assumed campaign lift rates in notebook 05, these campaigns could add about $475,000 in 12-month revenue. The lift rates are assumptions, not measured results. Real numbers would need A/B tests.

## Limitations

- **LTV is revenue, not profit.** The dataset has no cost or margin data.
- **Currency.** The source data is from a UK retailer, so prices are in British pounds. The notebooks format amounts with a `$` sign.
- **Campaign lift rates are assumptions.** The extra-revenue estimates in notebook 05 need A/B tests to confirm.
- **The largest buyers are understated.** Capping order values at the 95th percentile keeps the model stable but lowers predicted LTV for wholesale customers.
- **One-time buyers can't be scored for churn.** BG/NBD has no evidence about them yet, so they are handled as a separate segment.
- **Customers without an ID are excluded.** About 23% of invoice lines have no customer ID and can't be used for customer-level analysis.
- **The test window includes the holiday season,** which likely raises return rates compared with the rest of the year.
- **Segment cut-offs are judgment calls.** Different value and risk thresholds would move customers between segments.
- **Two years of data.** The model assumes customers keep behaving as they have. It does not account for promotions, price changes, or new products.

## Use of generative AI

Claude AI was used during this project for scoping the modeling approach, drafting and reviewing certain code in notebooks, and certain explanation text consolidation. All code was run and checked on the real dataset, and the findings and write-up were reviewed by the project team.