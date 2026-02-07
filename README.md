# Data Engineering – Moving Big Data Predict  
**Explore Data Science Academy** | February 2026

![AWS Data Pipeline Banner](https://example.com/placeholder-banner.png)  
*(Placeholder for a banner image representing AWS data pipeline architecture. Replace with a relevant image from AWS documentation or stock photos.)*

## Project Overview

This repository documents the implementation of an event-driven data pipeline for migrating stock market data from on-premises to AWS cloud. The pipeline handles batch processing of CSV files containing historical stock data for 1000 companies over 10 years, focusing on a subset for top companies.

As a data engineer, the goal was to build a robust ETL (Extract, Transform, Load) pipeline using Apache Airflow (containerized with Docker) on AWS EC2, integrating services like S3, RDS (PostgreSQL), SNS, and Lambda for automation and monitoring.

The pipeline is designed for:
- **Ingesting** up to 1000 small CSV files (<1MB each) per run.
- **Processing** and transforming data into a single CSV.
- **Loading** into a PostgreSQL table on RDS.
- **Notifying** success/failure via email (SNS).
- **Triggering** automatically on S3 file upload events (via Lambda).

No scaling assumptions are made, as per requirements (no concurrent runs, invocations ≤1/day).

### Key Technologies & Services
- **Orchestration**: Apache Airflow (Dockerized on EC2)
- **Storage**: Amazon S3 (source and monitored buckets)
- **Compute**: Amazon EC2 (t2.medium with custom AMI)
- **Database**: Amazon RDS (PostgreSQL db.t3.micro)
- **Notifications**: Amazon SNS
- **Automation**: AWS Lambda (event-triggered)
- **Processing**: Python (Pandas), SQL, Bash scripts
- **Mounting**: S3FS-Fuse for S3 access on EC2

## Functional Requirements & Implementation

The pipeline meets all specified requirements:

| Requirement | Description | Implementation |
|-------------|-------------|----------------|
| Pipeline Input | Ingest/process up to 1000 CSVs per run | S3 bucket with `Stocks/` folder; Airflow DAG handles batch processing |
| Pipeline Output | Store in single PostgreSQL table | RDS table `historical_stocks_data`; Loaded via `insert_query.sql` |
| Monitoring | Email success/failure notifications | SNS topic with subscriptions (personal + edsa.predicts@explore-ai.net); Subject lines: `FirstName_Surname_Pipeline_Success/Failure` |
| Automation | Event-driven on S3 file change | Monitored S3 bucket triggers Lambda on `.SUCCESS` file upload, which invokes Airflow DAG |
| Scaling | No assumptions needed | Single EC2 instance; No concurrency handling |

## Architecture Diagram

![Pipeline Architecture](https://example.com/placeholder-pipeline-diagram.png)  
*(Placeholder for Figure 1 from instructions. Create using tools like draw.io or Lucidchart based on Figures 1-5 descriptions: S3 → Lambda → Airflow/EC2 → Processing → RDS → SNS.)*

The pipeline layers include:
- **Security**: Custom security group, IAM role with S3/RDS/SNS full access.
- **Data Source**: Downloaded stock CSVs (7192 files) from Google Drive link, uploaded to S3 `Stocks/`.
- **Data Storage**: S3 source bucket with folders (`CompanyNames/`, `Output/`, `Scripts/`, `Stocks/`); RDS PostgreSQL.
- **Resources**: EC2 t2.medium with EDSA Airflow AMI (ID: ami-05f67f34b041e3b67).
- **Data Activities**: Python processing script (`process_stock_data.py`) from starter notebook; SQL insert script.
- **Data Communication**: SNS for alerts.
- **Automation**: Lambda layer with awscli; Triggered on monitored S3 bucket.

## Repository Structure

```
.
├── dags/                        
│   └── stock_etl_dag.py          # Airflow DAG script for orchestration
├── scripts/                     
│   ├── process_stock_data.py     # Python script for data transformation (converted from starter notebook)
│   ├── insert_query.sql          # SQL script for loading data into RDS
│   ├── mount_and_process.sh      # Bash script to mount S3 and run processing
│   └── top_companies.txt         # List of top companies (uploaded to S3)
├── lambda/                      
│   └── trigger_pipeline.py       # Lambda function code to invoke Airflow
├── docker/                      
│   ├── Dockerfile                # Custom Docker image for Airflow
│   └── docker-compose.yml        # Local testing setup
├── docs/                        
│   ├── architecture.drawio       # Editable diagram file (draw.io)
│   └── setup-guide.md            # Detailed setup notes
├── submission/                  
│   └── resource_details.csv      # CSV for marking (Name, Surname, Buckets, SNS, IP)
├── .gitignore                   
└── README.md                    # This file
```

## Setup & Configuration Guide

### Part I: Configure Pipeline Layers

1. **Security**:
   - Create security group with inbound rules for PostgreSQL (5432) from My IP and self, SSH from My IP.
   - Outbound: All traffic.
   - IAM Role for EC2: Attach `AmazonS3FullAccess`, `AmazonRDSDataFullAccess`, `AmazonSNSFullAccess`. Trust: ec2/rds.

2. **Data Source**:
   - Download stock dataset from [provided Google Drive link](https://example.com/stock-dataset) (placeholder; search for "stock market data 1000 companies 10 years CSV").
   - Unzip to 7192 CSVs.
   - Upload `top_companies.txt` to S3 `CompanyNames/`.

3. **Data Storage**:
   - Create source S3 bucket: `de-mbd-predict-{firstname}-{lastname}-s3-source`.
   - Folders: `CompanyNames/`, `Output/`, `Scripts/`, `Stocks/`.
   - Upload CSVs to `Stocks/` (use AWS CLI for efficiency).
   - RDS: PostgreSQL 14.6, db.t3.micro, 20GB, public access, default VPC, custom security group.

4. **EC2 Instance**:
   - AMI: edsa-airflow-ami (ID: ami-05f67f34b041e3b67).
   - Type: t2.medium, 30GB storage.
   - Key pair: Save `.pem` securely.
   - SSH connect, install Anaconda, Pandas, AWS CLI; Configure credentials.
   - Optional: Assign Elastic IP for static access.

5. **Mount S3**:
   - Install S3FS-Fuse on EC2.
   - Mount command: `s3fs -o iam_role=<role> -o url="https://s3.eu-west-1.amazonaws.com" -o endpoint=eu-west-1 -o allow_other <bucket> ~/s3-drive`.

6. **Data Processing**:
   - Convert starter notebook (from [here](https://example.com/starter-notebook)) to `process_stock_data.py`.
   - Output: `historical_stock_data.csv` in `Output/`.
   - Create bash script `mount_and_process.sh`.
   - Test locally, then on EC2.

7. **Database Table**:
   - Use pgAdmin (v5.5) to connect to RDS.
   - Create table `historical_stocks_data` with VARCHAR(56) fields (all nullable).
   - Create `insert_query.sql` for data loading.

8. **SNS**:
   - Topic: `de-mbd-predict-{First-name}-{Surname}-SNS`.
   - Subscriptions: Personal email + `edsa.predicts@explore-ai.net`.

### Part II: Pipeline Assembly

- Use Docker + Airflow on EC2 to build DAG.
- DAG tasks: Mount S3, process data, load to RDS, send SNS notification.
- Manual trigger for testing; Check Airflow UI logs and RDS table.

### Part III: Pipeline Automation

1. **Monitored Bucket**:
   - Name: `de-mbd-predict-{firstname}-{lastname}-monitored-bucket`.
   - Disable public block; Attach provided policy for marking.

2. **Lambda**:
   - Create function to trigger Airflow DAG.
   - Add layer for awscli.
   - Trigger: S3 `ObjectCreated` on `*.SUCCESS`.
   - Study and customize provided Lambda code.

### Submission
- Submit `resource_details.csv`:
  ```
  Name,Surname,Source_bucket,Monitored_bucket,SNS_topic,Static_IP
  YourName,YourSurname,de-mbd-predict-...,de-mbd-predict-...,de-mbd-predict-...,xx.xx.xx.xx
  ```
- Stop (don't terminate) EC2 after completion.

### Part V: Clean-Up
- Delete: Lambda, S3 buckets, RDS, EC2, SNS, Airflow resources.

## Potential Pitfalls & Cost Traps
- **Security**: Restrict PostgreSQL to My IP post-testing.
- **Costs**: Monitor Lambda/EC2/RDS; Use free tier; Stop instances.
- **Debugging**: Check CloudWatch, Airflow logs, SNS delays (5 mins).

## MCQs
- Complete on Athena: Data Pipeline MCQ, Python Processing MCQ.

## License & Acknowledgments
© Explore Data Science Academy. For educational purposes.  
Built using AWS best practices and Airflow documentation.  

If you have questions, check AWS docs or EDSA forums. Star the repo if useful! ⭐
