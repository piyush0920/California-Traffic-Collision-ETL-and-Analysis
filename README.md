# California Traffic Collision ETL & Analysis

A PySpark-based data preparation and exploratory analytics workflow for California traffic collision records. The project combines Spark DataFrame transformations, Spark SQL, and visual analysis to examine collision severity, contributing conditions, geography, and time patterns.

> **Project scope:** The committed notebook currently reads four staged CSV files. Although the notebook imports SQLite libraries and its source brief describes a SQLite dataset, the current extraction cells use CSV inputs rather than reading directly from a SQLite database. The input CSVs are not included in this repository.

## Project objectives

- Prepare and standardize collision, party, victim, and case identifier data.
- Handle missing values, invalid values, and selected outliers.
- Explore collision patterns by severity, weather, lighting, road conditions, county, and time.
- Use Spark SQL to answer operational questions about collision frequency and fatal outcomes.
- Demonstrate an optional S3 export step for the processed dataset.

## Data and expected inputs

The project is based on California traffic collision data described in the notebook as originating from the Statewide Integrated Traffic Records System (SWITRS).

Place these four prepared files in the notebook's working directory before running it:

| Expected file | Purpose |
|---|---|
| `sample_case_ids.csv` | Case identifier lookup / supporting case data |
| `sample_collisions.csv` | Collision-level attributes, including date, location, severity, and contributing factors |
| `sample_parties.csv` | Party-level attributes such as age, sex, sobriety, and vehicle information |
| `sample_victims.csv` | Victim-level attributes, including age and injury details |

The CSV files are not committed, so you must obtain the source data separately and prepare files with the expected column names and compatible schemas. Do not assume that the repository includes the original SQLite database or the raw SWITRS extracts.

## Technology stack

| Area | Tools |
|---|---|
| Data processing | Python, Apache Spark / PySpark 3.5.4 |
| Querying | Spark SQL |
| Data handling | pandas 2.2.2 |
| Visualization | Matplotlib, Seaborn |
| Storage extension | AWS S3 through Spark's S3A connector |
| Notebook environment | Jupyter / Google Colab |

## Workflow

### 1. Load and inspect the data

The notebook creates a Spark session and reads the four staged CSV files with schema inference enabled. It inspects schemas and sample rows before combining the datasets.

### 2. Join and prepare records

The current implementation uses left joins on `case_id` to combine collision, party, victim, and case identifier data.

**Important granularity note:** party and victim tables can contain multiple records for a collision. The notebook subsequently calls `dropDuplicates(["case_id"])`. That operation can collapse one-to-many records to a single row per case and may discard party- or victim-level detail. Review the intended analytical grain and aggregation logic before interpreting joined party/victim attributes as complete. A production version should preserve the natural keys or aggregate child records deliberately.

### 3. Clean and standardize

The notebook includes the following preparation steps:

- Drop selected sparse or low-value columns.
- Cast selected numeric fields to numeric types.
- Remove records missing values in a defined set of key analytical columns.
- Fill selected categorical fields with `Unknown` and selected numeric fields with sentinel values such as `-1` or `0`.
- Apply domain-based filtering to selected age and vehicle-year values.
- Use approximate IQR quantiles to inspect age outliers.
- Derive year, month, weekday, and hour fields.
- Rename and reorder fields for a more analysis-friendly output schema.
- Create a binary hit-and-run indicator.

The treatment of missing values and sentinel values is part of the notebook's current analysis workflow; those values should be considered explicitly before using the resulting table for statistical modelling.

### 4. Exploratory analysis and visualization

The notebook explores:

- Collision severity distribution.
- Weather conditions and collision severity.
- Victim and party age distributions.
- Severity versus the number of victims.
- Lighting conditions and collision severity.
- Weekday-wise and hourly collision trends.
- County-level collision distribution.
- Geographic patterns using latitude and longitude.
- Yearly and monthly collision trends.

### 5. Spark SQL analysis

After creating a temporary view named `collisions`, the notebook uses Spark SQL to investigate:

- The five counties with the highest collision counts.
- The month with the highest collision frequency.
- The most frequently recorded weather condition.
- The percentage of collisions involving fatalities.
- The hour with the highest collision count.
- Road surface and lighting conditions associated with the most recorded collisions.

These are descriptive associations in the supplied historical data; they should not be interpreted as causal effects or exposure-adjusted risk rates.

## Visual results

The repository includes the following charts generated from the notebook.

| Collision overview | Distribution and conditions |
|---|---|
| ![Collision severity](assets/chart_01.png) | ![Severity counts](assets/chart_02.png) |
| ![Road surface](assets/chart_03.png) | ![Lighting](assets/chart_04.png) |
| ![Collision type](assets/chart_05.png) | ![Party age](assets/chart_06.png) |
| ![Victim age](assets/chart_07.png) | ![Hit and run](assets/chart_08.png) |
| ![Primary factor](assets/chart_09.png) | ![Hourly distribution](assets/chart_10.png) |
| ![Weekday trends](assets/chart_11.png) | ![Monthly trends](assets/chart_12.png) |

## Run the notebook

### Prerequisites

- Python environment compatible with PySpark 3.5.4.
- A Java runtime compatible with the selected Spark distribution.
- The four prepared CSV input files listed above.

Install the main notebook dependencies:

```bash
pip install pyspark==3.5.4 pandas==2.2.2 matplotlib seaborn jupyter
```

Launch Jupyter:

```bash
jupyter notebook ETL-Project-Traffic-Collisions-Analysis-Unfinished.ipynb
```

Run the cells in order after placing the expected CSV files in the notebook's working directory. Some notebook paths and environment-specific settings may need to be adjusted for your machine or Colab session.

## Optional AWS S3 export

The notebook contains a Spark CSV write step targeting:

```text
s3a://collision_cleaned/processed/collisions_csv
```

Before running it, replace the destination with a bucket and prefix you control, configure the Hadoop S3A connector compatible with your Spark version, and provide AWS access through an IAM role or an approved credential mechanism. Do not hard-code AWS keys in the notebook or commit credentials to GitHub.

The S3 write step is an integration example; successful export depends on the user's Spark/AWS configuration.

## Repository structure

```text
.
├── ETL-Project-Traffic-Collisions-Analysis-Unfinished.ipynb
├── assets/
│   ├── chart_01.png
│   ├── chart_02.png
│   ├── ...
│   └── chart_12.png
└── README.md
```

## Limitations and next steps

- The four input CSV files and original source database are not included; data acquisition and preparation are separate prerequisites.
- The join-and-deduplication approach should be revisited to avoid unintentionally losing one-to-many party or victim records.
- The notebook collects data into pandas for several visualizations; large datasets may require aggregation in Spark before conversion to pandas.
- Input paths, schemas, and S3 settings should be parameterized for easier reuse.
- A pinned `requirements.txt`, refreshed notebook name, and a reproducible sample-data package would make the project easier to run.

## Author

**Piyush Sharma**  
Data Analytics · PySpark · Spark SQL · ETL  
[GitHub](https://github.com/piyush0920) · [Portfolio](https://piyush0920.github.io/)
