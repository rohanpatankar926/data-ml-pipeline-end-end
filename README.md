# Data & ML Pipeline - End-to-End Corporate Registry System

A comprehensive data engineering and machine learning pipeline for processing corporate registry data, featuring ETL workflows, entity resolution, schema harmonization, and ML-based profit prediction.

## 🏗️ Architecture Overview

This project implements a complete data pipeline system with the following components:

- **ETL Pipeline**: Extract, Transform, and Load corporate data from multiple sources
- **ML Pipeline**: Train and deploy machine learning models for profit prediction
- **Data Generation**: Tools for generating synthetic corporate data
- **Orchestration**: Apache Airflow for workflow management
- **Model Registry**: MLflow for experiment tracking and model versioning
- **Data Lake**: Apache Iceberg integration for scalable data storage

## 📋 Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Pipeline Details](#pipeline-details)
- [Development](#development)
- [Deployment](#deployment)

## ✨ Features

### ETL Pipeline
- **Multi-source Extraction**: Extract data from supply chain and financial data sources
- **Data Cleaning**: Automated data quality checks and cleaning
- **Entity Resolution**: Intelligent matching and merging of corporate entities across sources
- **Schema Harmonization**: Unified schema creation for corporate registry
- **Flexible Storage**: Support for S3 and Apache Iceberg table formats
- **Intermediate Checkpoints**: Optional saving of intermediate processing steps

### ML Pipeline
- **Feature Engineering**: Automated feature creation from corporate data
- **Model Training**: Logistic regression for profit threshold prediction
- **Model Registry**: MLflow integration for experiment tracking
- **Iceberg Integration**: Direct reading from Iceberg tables for training data

### Data Generation
- **Synthetic Data**: Generate realistic corporate data using Faker
- **Multiple Formats**: Support for Parquet, CSV, and JSON formats
- **S3 Upload**: Automated upload to S3 buckets

## 📦 Prerequisites

- **Docker** and **Docker Compose** (for containerized deployment)
- **Python 3.11+** (for local development)
- **AWS Account** with S3 access (for data storage)
- **Java 21** (for Spark/Iceberg support - included in Docker image)

## 📁 Project Structure

```
data-ml-pipeline-end-end/
├── dags/                      # Apache Airflow DAG definitions
│   ├── etl_pipeline_dag.py   # ETL workflow orchestration
│   ├── ml_pipeline_dag.py    # ML training workflow
│   └── airflow_utils.py      # Airflow configuration utilities
├── etl_pipeline/             # ETL pipeline modules
│   ├── extract.py            # Data extraction from S3
│   ├── transform.py          # Data transformation and cleaning
│   ├── load.py               # Data loading to S3/Iceberg
│   └── s3_client.py          # S3 client utilities
├── ml_pipeline/              # ML pipeline modules
│   ├── pipeline.py           # Main ML pipeline orchestration
│   ├── data_reader.py        # Iceberg data reading
│   ├── feature_engineering.py # Feature creation
│   ├── model_training.py     # Model training logic
│   └── model_registry.py     # MLflow integration
├── data_upsert/              # Data generation tools
│   ├── main_data_generate_pipeline.py # Main data generation script
│   ├── data_generator.py     # Synthetic data generation
│   └── s3_uploader.py        # S3 upload utilities
├── docker-compose.yml        # Docker Compose configuration
├── Dockerfile                # Docker image definition
├── requirements.txt          # Python dependencies
└── entrypoint.sh            # Airflow initialization script
```

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd data-ml-pipeline-end-end
```

### 2. Set Up Environment Variables

Create a `.env` file in the project root:

```bash
# AWS Configuration
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_REGION=us-east-1
AWS_ACCOUNT_ID=your-account-id

# S3 Configuration
S3_INPUT_BUCKET=your-input-bucket
S3_OUTPUT_BUCKET=your-output-bucket
S3_SOURCE1_PATH=source1_supply_chain.parquet
S3_SOURCE2_PATH=source2_financial.parquet
S3_OUTPUT_PATH=processed/

# ETL Configuration
ETL_OUTPUT_FORMAT=parquet
SAVE_INTERMEDIATE=false

# Iceberg Configuration (Optional)
ICEBERG_ENABLED=false
ICEBERG_TABLE_PATH=corporate_db.corporate_registry
ICEBERG_S3_LOCATION=s3://your-bucket/iceberg/
ICEBERG_PARTITION_KEY=date
ICEBERG_DATABASE_NAME=corporate_db
ICEBERG_TABLE_NAME=corporate_registry

# ML Configuration
PROFIT_THRESHOLD=100.0
TRAIN_RATIO=0.8
MAX_ITER=100
REG_PARAM=0.01
FEATURE_COLUMNS=revenue,num_suppliers,profit_margin_normalized
MODEL_NAME=corporate_profit_predictor
MLFLOW_EXPERIMENT_NAME=corporate_profit_prediction
```

### 3. Generate Sample Data (Optional)

If you need to generate sample data for testing:

```bash
cd data_upsert
python main_data_generate_pipeline.py 1000 --config pipeline_config.json
```

### 4. Start Services with Docker Compose

```bash
docker-compose up -d
```

This will start:
- **Airflow Webserver** on `http://localhost:8080`
- **Airflow Scheduler** (background service)
- **MLflow Server** on `http://localhost:5000`

### 5. Access the Services

- **Airflow UI**: http://localhost:8080
  - Username: `admin`
  - Password: `admin`
- **MLflow UI**: http://localhost:5000

### 6. Trigger Pipelines

1. Navigate to Airflow UI
2. Enable the DAGs:
   - `etl_corporate_registry_pipeline` (runs daily)
   - `ml_corporate_profit_prediction` (runs weekly)
3. Trigger manually or wait for scheduled execution

## ⚙️ Configuration

### Environment Variables

The pipeline supports configuration through environment variables (see `.env` file above) or Airflow Variables.

### Airflow Variables

You can also set configuration via Airflow UI:
1. Go to Admin → Variables
2. Create a JSON variable named `etl_pipeline_config` with your configuration

### Data Formats

The pipeline supports multiple output formats:
- **Parquet** (recommended): Efficient, compressed, preserves data types
- **CSV**: Human-readable, universal compatibility
- **JSON**: Preserves structure, good for APIs

## 🔄 Pipeline Details

### ETL Pipeline Workflow

The ETL pipeline (`etl_corporate_registry_pipeline`) consists of three main tasks:

1. **Extract**
   - Reads supply chain data (`source1_supply_chain.parquet`) from S3
   - Reads financial data (`source2_financial.parquet`) from S3
   - Saves intermediate extracted data

2. **Transform**
   - Cleans and validates data from both sources
   - Resolves entities (matches corporations across sources)
   - Harmonizes schema to create unified corporate registry
   - Creates final corporate registry dataset

3. **Load**
   - Saves corporate registry to S3
   - Optionally upserts to Apache Iceberg table (if enabled)

**Schedule**: Daily at midnight UTC

### ML Pipeline Workflow

The ML pipeline (`ml_corporate_profit_prediction`) performs:

1. **Data Reading**: Reads corporate registry from Iceberg table
2. **Feature Engineering**: Creates features for profit prediction
3. **Model Training**: Trains logistic regression model
4. **Model Registration**: Registers model with MLflow

**Schedule**: Weekly (every 7 days)

**Model**: Logistic Regression for binary classification (profit above/below threshold)

**Features**: 
- Revenue
- Number of suppliers
- Profit margin (normalized)
