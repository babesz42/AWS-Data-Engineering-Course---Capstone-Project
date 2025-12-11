# AWS-Data-Engineering-Course---Capstone-Project - Markdown
**Author:** Szabó Bence
This is a documentation about the Capstone projec of the AWS data engineering course for my Large-Scale Data Architectures class in university.

## 1. Goal of the Project
The primary objective of this project is to architect and implement a scalable serverless data pipeline on AWS to ingest, transform, and analyze historical global fisheries data (1950–2018) from the "Sea Around Us" research initiative.

This project simulates a real-world scenario where the infrastructure must support data analysts in generating reports on fishing impacts in Open Seas and Exclusive Economic Zones (EEZs). Specifically, the project aims to:

1.  **Ingest & Store:** Securely host raw CSV data containing catch tonnage and economic value (in 2010 USD) for various fishing areas, including the Pacific Western Central Open Seas and Fiji's EEZ.
2.  **Transform:** Convert raw CSV files into the column-oriented Apache Parquet format to optimize storage costs and query performance.
3.  **Catalog:** Automate schema discovery using AWS Glue Crawlers to populate the AWS Glue Data Catalog.
4.  **Analyze:** Enable ad-hoc SQL analysis using Amazon Athena to extract insights regarding catch trends, economic value, and fishing entities without managing underlying infrastructure.

By completing this pipeline, I demonstrate the ability to process large-scale environmental datasets and prepare them for downstream analytics and reporting.

# 2. Steps Taken

## 2.1 Set up development enviroment

### 2.1.1 Create an AWS Cloud9 environment

### 2.1.2 Create an AWS Cloud9 environment
- data-source-6769
- query-result-6769

### 2.1.3 Download the three .csv source files
Run this in AWS Cloud9 IDE:

`wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACDENG-1-91570/lab-capstone/s3/SAU-GLOBAL-1-v48-0.csv`

`wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACDENG-1-91570/lab-capstone/s3/SAU-HighSeas-71-v48-0.csv`

`wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACDENG-1-91570/lab-capstone/s3/SAU-EEZ-242-v48-0.csv`

### 2.1.4 Check data
There is more columns that it says in the lab, but I have those that we should have

The number of lines is okay
`wc -l SAU-GLOBAL-1-v48-0.csv`

### 2.1.5 Convert the SAU-GLOBAL-1-v48-0.csv file to Parquet format

### 2.1.6 Upload `SAU-GLOBAL-1-v48-0.parquet` to `data-source-6769` bucket
`aws s3 cp SAU-GLOBAL-1-v48-0.parquet s3://data-source-6769/`

## 2.2 Using an AWS Glue crawler and querying multiple files with Athena
Configure an AWS Glue crawler to discover the structure of the data and then use Athena to query the data.

### 2.2.1 Convert the `SAU-HighSeas-71-v48-0.csv` file to Parquet format and upload it to the `data-source-6769` bucket.
```
python3
import pandas as pd
df = pd.read_csv('SAU-HighSeas-71-v48-0.csv')
df.to_parquet('SAU-HighSeas-71-v48-0.parquet')
exit()
```

bash: `aws s3 cp SAU-HighSeas-71-v48-0.parquet s3://data-source-6769/`

--> Both files are in the S3 bucker

### 2.2.2 Create an AWS Glue database and an AWS Glue crawler

### 2.2.3 Run the crawler to create a table that contains metadata in the AWS Glue database

### 2.2.4 To confirm that the table properly categorized the data, use Athena to run SQL queries against each column in the new table
`SELECT DISTINCT area_name FROM fishdb.data_source_6769;` --> works

Next:
```
SELECT year, fishing_entity AS Country, CAST(CAST(SUM(landed_value) AS DOUBLE) AS DECIMAL(38,2)) AS ValuePacificWCSeasCatch
FROM fishdb.data_source_6769
WHERE area_name LIKE '%Pacific%' and fishing_entity='Fiji' AND year > 2000
GROUP BY year, fishing_entity
ORDER By year
```
--> also works

Next:

```
SELECT year, fishing_entity AS Country, CAST(CAST(SUM(landed_value) AS DOUBLE) AS DECIMAL(38,2)) AS ValueAllHighSeasCatch
FROM fishdb.data_source_6769
WHERE area_name IS NULL and fishing_entity='Fiji' AND year > 2000
GROUP BY year, fishing_entity
ORDER By year
```
--> created view from query with name `challange`
