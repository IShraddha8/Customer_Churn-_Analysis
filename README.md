# Customer Churn Analysis

An end-to-end data and machine learning project that identifies customers at risk of churning and surfaces actionable retention insights. The pipeline runs from raw data ingestion in SQL Server, through exploratory analysis in Power BI, into an XGBoost prediction model, and finally out through a live Streamlit dashboard that any business user can operate.

The dataset comes from a telecom context, but nothing in this architecture is domain-specific. Swap the CSV for banking transactions, SaaS usage logs, or e-commerce purchase histories and the same pipeline runs.

<br>

## What This Project Does

Takes raw customer data and answers two questions: who has already churned and why, and which new customers are likely to churn next. The first question is answered through Power BI. The second is answered through a trained XGBoost model deployed in a Streamlit app called ChurnRadar.

At the historical analysis level, 6,418 customers were analyzed with an overall churn rate sitting at 27%. On the prediction side, the model flagged 388 of 411 new joiners as at-risk, representing ₹44,512 in revenue that the retention team now has a chance to protect.

<br>



## Key Numbers

| Metric | Value |
|---|---|
| Total Customers | 6,418 |
| New Joiners | 411 |
| Historical Churn Count | 1,732 |
| Historical Churn Rate | 27.0% |
| Predicted At-Risk Customers | 388 |
| Revenue at Risk | ₹44,512 |
| Model Accuracy | 83.86% |
| Model Recall (Churn Class) | 73.78% |

<br>

## What Drives Churn

Month-to-Month contract customers churn at 46.5%, compared to 11% for One Year and 2.7% for Two Year contracts. That gap alone tells most of the story.

Beyond contract type, Fiber Optic internet users churn at 57.9%, which is abnormally high and points to either service reliability issues or a pricing problem relative to competitors. The biggest single churn category is competitor switching, with 761 customers explicitly citing that as their reason for leaving.

Geographically, Jammu & Kashmir (57.2%), Assam (38.1%), and Jharkhand (34.5%) have the highest churn rates by state. On the prediction side, Uttar Pradesh (45 flagged), Maharashtra (41), and Tamil Nadu (36) need the most immediate retention attention.

Customers who subscribe to security and support add-ons churn significantly less. Online Security non-subscribers churn at 84.6% versus 15.4% for subscribers. That pattern repeats across Premium Support, Online Backup, and Device Protection Plan.

<br>

## Project Structure

```
Customer Churn Analysis Project/
│
├── Data & Resources/
│   └── Customer_Data.csv
│
├── Machine Learning Predictions/
│   ├── Churn_Predictions.ipynb
│   ├── churn_preprocessor.joblib
│   ├── baseline_xgb_model.joblib
│   ├── high_risk_churn_list.csv
│   └── new_joiners_data.csv
│
├── Power BI Dashboard Screenshots/
│   ├── Summary_Page.png
│   └── Predictions_Page.png
│
├── SQL/
│   ├── 01_etl_pipeline.sql
│   ├── 02_kpi_analysis.sql
│   ├── 03_data_quality_check.sql
│   ├── 04_imputation_strategy.sql
│   ├── 05_clean_table.sql
│   └── 06_analytical_views.sql
│
├── Streamlit App/
│   └── churn_prediction.py
│
├── Churn Analysis.pbix
├── Power BI Dashboard.pdf
└── Power BI Report - Insights.pdf
```

<br>

## Tech Stack

| Layer | Tool |
|---|---|
| Data Storage | Microsoft SQL Server (T-SQL) |
| Business Intelligence | Power BI Desktop |
| Language | Python 3.12 |
| ML Libraries | scikit-learn, XGBoost, LightGBM |
| Data Handling | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Live Dashboard | Streamlit |
| Model Persistence | joblib |
| DB Connector | SQLAlchemy + pyodbc |

<br>

## How the Pipeline Works

**SQL Server** handles everything from raw data ingestion to producing clean, analysis-ready views. The pipeline runs across six scripts. Raw CSV data lands in a staging table first (`dbo.stg_Churn`). A null audit runs across all 32 columns. Missing values in service columns are filled with `No` or `None` rather than dropped, because a null there means the customer simply doesn't subscribe, it's not missing data. Churn classification nulls go into an `Others` bucket. The cleaned data writes into a production table (`dbo.prod_Churn`), and two SQL views split it by customer status: `vw_ChurnData` for historical training data, `vw_JoinData` for new joiners.

**Power BI** connects directly to those views. No manual CSV exports needed for the BI layer. The two-page report includes slicers for Monthly Charge Range and Marital Status that filter across all visuals simultaneously.

**The Jupyter Notebook** pulls from SQL Server using SQLAlchemy, runs preprocessing through a `ColumnTransformer` pipeline (OrdinalEncoder for Contract, OneHotEncoder for the remaining 18 categorical columns, passthrough for numerics), trains three models, and serializes the winner.

**Three models were compared:**

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Random Forest | 0.8378 | 0.7123 | 0.7349 | 0.7234 |
| XGBoost | 0.8386 | 0.7131 | 0.7378 | 0.7252 |
| LightGBM | 0.8228 | 0.6642 | 0.7810 | 0.7179 |

