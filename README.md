# Final project for MSBA 510 (Python for Business Analytics) at CSUCI (Working Draft)
Ingests UK online retail transactions (2009–2011), cleans them through bronze, silver, and gold layers in DuckDB, and models customer lifetime value and churn risk in Jupyter notebooks. Customer segments feed campaign recommendations. 

Data source: UCI Machine Learning Repository, Online Retail II.
Dataset link: https://archive.ics.uci.edu/dataset/502/online+retail+ii

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
-**Notebook 02 seems stuck:** it stops and waits for you to type `yes` or `no` before saving. Type `yes` in the box at the top of the window and press Enter.