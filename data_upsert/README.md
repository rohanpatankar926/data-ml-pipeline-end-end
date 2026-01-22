# Data Generation and S3 Upload Pipeline

Automated pipeline to generate dummy data and upload to S3 bucket.

## Quick Start

```bash
# Generate 100 rows and upload to S3 (using config file)
python main_data_generate_pipeline.py 100 --config pipeline_config.json

# Or with command line arguments
python main_data_generate_pipeline.py 100 --bucket my-bucket --format parquet
```

## Data Sources

- **Source 1 (Supply Chain Data)**: Corporations with `corporate_name_S1`, `address`, `activity_places`, and `top_suppliers`
- **Source 2 (Financial Data)**: Corporations with `corporate_name_S2`, `main_customers`, `revenue`, and `profit`

## Installation

Install the required dependencies:

```bash
pip install faker pandas pyarrow boto3
```

## Main Pipeline

The `main_data_generate_pipeline.py` script automates the entire workflow:

### Basic Usage

```bash
# Generate and upload with config file
python main_data_generate_pipelineeline.py 100 --config pipeline_config.json

# Generate only (skip upload)
python main_data_generate_pipelineeline.py 100 --no-upload

# With command line overrides
python main_data_generate_pipeline.py 100 --bucket my-bucket --format parquet --output-dir ./data
```

### Configuration File

Create `pipeline_config.json`:

```json
{
  "generation": {
    "num_rows": 100,
    "output_format": "parquet",
    "output_dir": "./data"
  },
  "s3": {
    "bucket": "your-bucket-name",
    "source1_path": "source1_supply_chain.parquet",
    "source2_path": "source2_financial.parquet"
  },
  "aws": {
    "access_key_id": "your-access-key-id",
    "secret_access_key": "your-secret-access-key",
    "region": "us-east-1"
  },
  "upload": {
    "enabled": true,
    "skip_source1": false,
    "skip_source2": false
  }
}
```

### Command Line Options

```bash
python main_data_generate_pipeline.py [NUM_ROWS] [OPTIONS]

Positional Arguments:
  NUM_ROWS              Number of rows to generate (optional if in config)

Options:
  --config PATH         Path to configuration JSON file
  --format FORMAT       Output format: json, csv, parquet, or both (default: parquet)
  --output-dir DIR      Output directory for generated files (default: ./data)
  --bucket BUCKET      S3 bucket name
  --no-upload           Skip S3 upload step
  --skip-source1        Skip uploading source1 file
  --skip-source2        Skip uploading source2 file
```

### Examples

```bash
# Simple usage with config
python main_data_generate_pipeline.py 100 --config pipeline_config.json

# Generate only, no upload
python main_data_generate_pipeline.py 50 --no-upload

# Override config with command line
python main_data_generate_pipeline.py 200 --bucket my-bucket --format csv

# Generate and upload specific format
python main_data_generate_pipeline.py 1000 --format parquet --output-dir ./output
```

## Using Environment Variables

You can also configure using environment variables:

```bash
export NUM_ROWS=100
export OUTPUT_FORMAT=parquet
export OUTPUT_DIR=./data
export S3_BUCKET=my-bucket
export AWS_ACCESS_KEY_ID=your-access-key
export AWS_SECRET_ACCESS_KEY=your-secret-key
export AWS_REGION=us-east-1
export UPLOAD_TO_S3=true

python main_data_generate_pipeline.py
```

## Output Files

### Generated Files

- `source1_supply_chain.parquet` - Supply Chain Data (default)
- `source2_financial.parquet` - Financial Data (default)
- `source1_supply_chain.csv` - Supply Chain Data (if CSV format)
- `source2_financial.csv` - Financial Data (if CSV format)
- `source1_supply_chain.json` - Supply Chain Data (if JSON format)
- `source2_financial.json` - Financial Data (if JSON format)

## Data Structure

### Source 1 (Supply Chain Data)

- `corporate_name_S1`: Company name
- `address`: Full address
- `activity_places`: List of activity locations (2-5 items)
- `top_suppliers`: List of supplier names (3-8 items)

### Source 2 (Financial Data)

- `corporate_name_S2`: Company name
- `main_customers`: List of customer names (2-6 items)
- `revenue`: Revenue in millions (10M - 5000M)
- `profit`: Profit in millions (5-20% of revenue)

## Format Comparison

| Format      | Pros                                        | Cons                               | Best For                  |
| ----------- | ------------------------------------------- | ---------------------------------- | ------------------------- |
| **Parquet** | Efficient, preserves data types, compressed | Requires pandas/pyarrow            | Production, ETL pipelines |
| **CSV**     | Human-readable, universal                   | Loses list structure, larger files | Manual inspection, Excel  |
| **JSON**    | Preserves structure, readable               | Larger files, slower               | APIs, web applications    |

## Module Structure

The code is organized into reusable modules:

- **`data_generator.py`** - Data generation functions
- **`s3_client.py`** - Common S3 client utilities
- **`s3_uploader.py`** - S3 upload functions
- **`main_data_generate_pipeline.py`** - Main pipeline script (entry point)

## Notes

- **Parquet format** is recommended for ETL pipelines as it:
  - Preserves list/array data types natively
  - Provides better compression
  - Is faster to read/write
  - Maintains data types (no type inference needed)
- **CSV format** converts lists to semicolon-separated strings
- **JSON format** preserves the full structure but is less efficient for large datasets
