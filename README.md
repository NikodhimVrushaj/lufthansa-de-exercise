# Data Engineering Exercise with PySpark, Delta Lake and Jupyter Notebooks

This exercise focus on an data pipeline using pyspark in jupyter notebooks, focusing on the medallion architecture (bronze, silver, gold) with delta tables.
The data for this exercise is retrieved from brazilian e-commerce dataset by Olist.

The pipeline follows the medallion architecture:
- Bronze: csv ingestion into delta tables,
- Silver: cleaning, transformation, enrichment, and calculated columns,
- Gold: analytical views, KPIs, functions, and reportings.

Addition :
- Bonus: reusable pyspark pipeline


## Project Structure

lufthansa-de-exercise/
├── notebooks/
│   ├── 01_bronze_ingestion.ipynb
│   ├── 02_silver_transformation.ipynb
│   ├── 03_gold_analytics.ipynb
│   └── 04_reusable_pipeline.ipynb
├── data/
│   └── CSV files
├── delta/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── delta_bonus/
│   └── processed/
├── README.md
├── requirements.txt
└── .gitignore


## Main Technologies

- Python 3.11.9
- Java 17
- PySpark
- Delta Lake
- Jupyter Notebook
- Pandas and PyArrow for optional local inspection
- Git and GitHub

Note: 
Since i had compatibility problems with these pyspark and delta-spark versions using them together with newer python version, i built this project and tested on:
- python 3.11.9 (i would recommend using this version)
- java 17 version required also, which supports spark 3.5.
 

## Notebook execution

The notebooks must be run at this order:

01_bronze_ingestion.ipynb
02_silver_transformation.ipynb
03_gold_analytics.ipynb
04_bonus_pipeline.ipynb


## Raw Data

The project ucontains these files:
olist_customers_dataset.csv
olist_orders_dataset.csv
olist_order_items_dataset.csv
olist_order_payments_dataset.csv
olist_order_reviews_dataset.csv
olist_products_dataset.csv
olist_sellers_dataset.csv


# Pipeline section

## Bronze Layer

(tables without a date column are not partitioned)

- loads CSV files into dataframes,
- parses timestamp columns and derives year, month, and day,
- partitions by the derived columns,
- writes as Delta tables.

## Silver Layer

- loads bronze delta tables,
- perform data cleaning from null values and duplication,
- apply transformation functions,
- joins between datasets for enrichment,
- derives calculates columns: Total Price, Delivery Time (days), Payment Count, Profit Margin,
- writes delta tables into silver layer maintaing partitions.

## Gold Layer

- create analytical views: cumulative sales per customer, rolling average delivery time per product category
- create kpi tables: total sales per product category, average delivery time per seller, order counts per customer state
 writes delta tables into gold layer maintaining partitions,
- provide insights from query results.

### Key Insights

Insights regarding product category:
Rank 1: beleza_saude generated 1,441,248.07, making up 9.10% of total sales.
Rank 2: relogios_presentes generated 1,305,541.61, making up 8.24% of total sales.
Rank 3: cama_mesa_banho generated 1,241,681.72, making up 7.84% of total sales.

Insights regarding customer state: 
1. SP: 41,375 orders, 41.93% of all orders.
2. RJ: 12,762 orders, 12.93% of all orders.
3. MG: 11,544 orders, 11.70% of all orders.
4. RS: 5,432 orders, 5.51% of all orders.
5. PR: 4,998 orders, 5.07% of all orders.

Insights regarding seller delivery: 
Fastest seller with at least 20 orders: 41c2bad7229b0c25e6becf179ebf63ff at 4.5 days.
Slowest seller with at least 20 orders: 66e0557ecc2b4dbea057e93f215f68d8 at 31.63 days.


## Reusable Pipeline

- csv and delta loading,
- handling null values and deduplication,
- apply transformations through F.col(), F.expr(), and F.when(),
- perform running totals and rolling averages using Window.partitionBy().orderBy(),
- timestamp parsing using  F.to_timestamp() before extracting year, month, and day.
- write partitioned Delta Tables in regards to the derived columns.


## Before insatalling packages

### Install Java 17

Java 17 is supported by Spark 3.5: 
https://adoptium.net/temurin/releases/?version=17

echo $env:JAVA_HOME

### Configure Hadoop Windows utilities

The local Windows setup used:

C:\hadoop
└── bin
    ├── winutils.exe
    └── hadoop.dll

Set:

$env:HADOOP_HOME = "C:\hadoop"
$env:PATH = "C:\hadoop\bin;$env:PATH"


## Setup on Windows

### 1. Clone the repository

git clone https://github.com/NikodhimVrushaj/lufthansa-de-exercise.git
cd lufthansa-de-exercise

### 2. Create and activate a virtual environment

python -m venv .venv
.\.venv\Scripts\Activate.ps1

### 3. Install Python packages

python -m pip install --upgrade pip
python -m pip install -r requirements.txt

### 4. Start Jupyter

jupyter notebook

- Use kernel Python 3.11(lufthansa-de)


## Problems that happened during execution and their solutions

- PowerShell blocked activation of venv activation

Solution: Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

- Jupyter used the wrong python installation, python worker crashes

Solution:
before creating SparkSession executed:

import os
import sys

os.environ["PYSPARK_PYTHON"] = sys.executable
os.environ["PYSPARK_DRIVER_PYTHON"] = sys.executable


- spark failed to start or execute jobs, while showing erros like:
java errors,spark failures and Hadoop errors.

Solution:
ensure these files exist:
C:\hadoop\bin\winutils.exe
C:\hadoop\bin\hadoop.dll

current hadoop version used is 3.3.4. Compatible with 3.3.6