# Final project for MSBA 510 (Python for Business Analytics) at CSUCI
Ingests UK online retail transactions (2009–2011), cleans them through bronze, silver, and gold layers in DuckDB, and models customer lifetime value and churn risk in Jupyter notebooks. Customer segments feed campaign recommendations. 

Data source: UCI Machine Learning Repository, Online Retail II.

```online-retail-ltv-churn-medallion/
  data/raw/          # downloaded file lands here
  data/ltv.duckdb    # bronze/silver/gold tables
  notebooks/
    01_ingest_bronze.ipynb
    02_silver_clean.ipynb
    03_gold_rfm_cohorts.ipynb
    04_ltv_churn_model.ipynb
    05_segments_recs.ipynb
  requirements.txt
  README.md```