# telco-churn-pipeline-nifi-hive-spark-hbase
# Telco Customer Churn Prediction Pipeline

End-to-end big data pipeline for customer churn prediction using NiFi ingestion, Hive, PySpark MLlib, and HBase storage.

## Overview

This project builds a complete data pipeline that ingests, stores, models, and persists results for a customer churn prediction use case, using a Hadoop-based big data stack:

**NiFi → HDFS → Hive → PySpark (MLlib) → HBase**

Raw customer data is ingested via Apache NiFi, stored in HDFS, structured into a Hive table for querying, used to train a logistic regression churn model in PySpark, and the resulting evaluation metrics are written back into HBase for persistence and future comparison across runs.

## Dataset

[Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) (Kaggle) — 7,043 customer records with demographic info, account details, and service subscriptions, with a binary churn label.

## Pipeline

**1. Data Ingestion (NiFi → HDFS)**
Three NiFi processors handle ingestion: `InvokeHTTP` downloads the CSV from GitHub, `UpdateAttribute` renames it to `telco_churn.csv`, and `PutHDFS` writes it to `/data/final_project` in HDFS.

**2. Data Warehousing (Hive)**
A managed Hive table (`final_project.telco_churn`) is built over the ingested data, with column types matched to the source schema (STRING for categorical fields, INT for tenure/SeniorCitizen, DOUBLE for charges). Verified via a `GROUP BY` on churn (5,174 non-churn / 1,869 churn).

**3. Environment Setup**
`numpy` and `happybase` installed on the Spark master and worker nodes so PySpark can write results to HBase via the Thrift server.

**4. Metrics Store (HBase)**
An HBase table (`telco_metrics`) with column family `cf` stores evaluation metrics per model run, keyed by a unique run ID.

**5. Model Training (PySpark MLlib)**
`telco_churn_lr.py` loads data from Hive, cleans and casts fields, encodes categorical variables (`StringIndexer` + `OneHotEncoder`), assembles a feature vector, and trains a Logistic Regression model on a 70/30 train-test split.

**6. Evaluation & Persistence**
Model performance is evaluated using AUC, accuracy, F1, weighted precision, and weighted recall, then written to HBase under a timestamped run ID.

## Results

| Metric | Score |
|---|---|
| AUC (ROC) | ~0.84 |
| Accuracy | ~80% |

## Tech Stack

Apache NiFi · HDFS · Hive · PySpark (MLlib) · HBase · HappyBase · Python

## Repository Contents

- `telco_churn_lr.py` — PySpark ML pipeline: data loading, feature engineering, model training, evaluation, and HBase write-back.
