# Air Quality Analysis using Spark


## Section 1: Data Ingestion and Initial Pre-Processing

✅ Objective

In this section, we simulate a real-time air quality data pipeline by:

Preprocessing the UCI Air Quality dataset

Simulating sensor data streaming via a TCP server

Ingesting and structuring the data with Spark Structured Streaming

Writing clean, timestamped outputs to CSV files for future processing

## 🗂 Folder Structure After This Section

air_quality_analysis_spark/
├── ingestion/
│   ├── data/
│   │   ├── pending/                 # Original + preprocessed input files
│   │   │   └── prepared/            # Streamable mini-batches
│   │   └── processed/              # Files that were streamed
│   ├── preprocess_airquality.py    # Prepares batch files from AirQualityUCI.csv
│   ├── tcp_log_file_streaming_server.py  # TCP streaming server
│   └── spark_streaming_ingestion.py      # Spark Structured Streaming client
└── section1/
    └── output/
        ├── clean_data_csv/         # Final structured outputs (part-*.csv)
        └── checkpoint_dir_csv/     # Spark checkpoint files


## 🧩 Step-by-Step Execution

### Step 1: Preprocess the UCI Dataset

To generate multiple streaming mini-batches, we included an external dataset:

Path: ingestion/data/pending/AirQualityUCI.csv

This dataset was preprocessed using preprocess_airquality.py to simulate realistic sensor data in a streamable format. The resulting batch files are saved in ingestion/data/pending/prepared/

Converts AirQualityUCI.csv into small, streamable CSV files

```bash
python ingestion/preprocess_airquality.py
```

#### 📁 Output:
Creates batch files under ingestion/data/pending/prepared/.

### Step 2: Start the TCP Server
Streams the batch files line-by-line with simulated delay.

``` bash
python ingestion/tcp_log_file_streaming_server.py
```
#### 💡 This simulates real-time sensor data over port 9999.

### Step 3: Start Spark Streaming Client

Reads from TCP, structures the data, and writes clean outputs

```bash
python ingestion/spark_streaming_ingestion.py
```

#### 📁 Output:
Creates CSVs under section1/output/clean_data_csv/
(Each file represents a mini-batch like part-00000.csv, part-00001.csv, ...)


## 📑 Output CSV Sample Schema

| timestamp              | region  | PM2_5 | temperature | humidity |
|------------------------|---------|-------|-------------|----------|
| 2004-03-10T18:00:00Z   | Region1 | 2.5   | 13.4        | 48.2     |
| 2004-03-10T19:00:00Z   | Region1 | 3.1   | 13.0        | 47.5     |


# Section 2: Data Aggregations, Transformations & Trend Analysis

## Objective

This module enhances the raw air quality dataset by addressing data quality issues (e.g., missing values, outliers), applying feature normalization, and generating temporal trend features. The processed data is suitable for statistical analysis, Spark SQL queries, and machine learning tasks.

---

## Key Tasks Performed

### 1. **Handling Missing Values**
- Imputed missing values in `PM2_5`, `temperature`, and `humidity` using mean imputation.
- Ensures the dataset remains complete and suitable for downstream analytics.

### 2. **Handling Outliers**
- Outliers in `PM2_5`, `temperature`, and `humidity` were capped at `mean ± 3 * stddev`.
- This preserves natural variation while preventing distortions caused by extreme values.

### 3. **Normalization**
- Applied Z-score normalization on key features:
  - `PM2_5_zscore`
  - `temperature_zscore`
  - `humidity_zscore`
- Normalized fields allow for consistent comparison and improved model convergence.

### 4. **Time-Based Aggregations**
- Generated:
  - **Daily aggregates**: Average PM2.5, temperature, and humidity by date.
  - **Hourly aggregates**: Average values by hour and date.
- Useful for identifying macro- and micro-trends over time.

### 5. **Lag Features & Rate of Change**
- Added:
  - `PM2_5_lag1`: Previous PM2.5 value (1-step lag).
  - `PM2_5_rate_change`: Percentage change from the previous reading.
- Enhances temporal modeling and supports trend detection in ML tasks.

---

## Output Structure

- `section2/output/feature_engineered_data/`: Fully processed dataset with lag features and normalization.
- `section2/output/daily_aggregations/`: Daily averages of key metrics.
- `section2/output/hourly_aggregations/`: Hourly averages of key metrics.

All outputs are stored in CSV format with headers.

---

## How to Run

1. Ensure Spark is installed and configured.
2. Place cleaned CSV files in: `section1/output/clean_data_csv/`.
3. Run the script:  
   ```bash
   spark-submit data_transformations_section2.py
