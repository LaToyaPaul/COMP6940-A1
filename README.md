## COMP6940 A1 — NYC Yellow Taxi Analysis (Part 1, Option A)

This project analyzes **NYC Yellow Taxi trips for January 2023**, focusing on demand, pricing, and temporal patterns using the official NYC TLC trip record data.

The work is organized into three main notebooks under `part1_taxi` and a final written report.

---

## Data Sources

We use the following official NYC TLC data and documentation (exact files required by the assignment):

- **Yellow taxi trip records (January 2023, Parquet)**  
  - [Link to the data](https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet)  
  - Saved locally as: `data/raw/tlc/yellow_tripdata_2023-01.parquet`

- **Taxi zone lookup (CSV)**  
  - [Link to the data](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv)  
  - Saved locally as: `data/raw/lookup/taxi_zone_lookup.csv`

- **Official TLC trip record documentation** (schema and definitions)  
  - [Link to the documentation](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)  
  - Used to interpret column meanings and valid value ranges.

---

## Project Structure

- **`part1_taxi/01_ingest.ipynb`**  
  Data download, storage, and basic inspection.

- **`part1_taxi/02_clean_features.ipynb`**  
  Data cleaning, feature engineering, and creation of the curated dataset.

- **`part1_taxi/03_stats_eda_viz.ipynb`**  
  Statistical summaries, groupby and window metrics, EDA to answer specific questions, and visualization.

- **`data/raw/`**  
  - `tlc/yellow_tripdata_2023-01.parquet` — raw January 2023 trip records.  
  - `lookup/taxi_zone_lookup.csv` — taxi zone metadata.

- **`data/curated/`**  
  - `part1_taxi_curated.parquet` — cleaned and feature-engineered dataset.

- **`part1_taxi/report.pdf`**  
  Final report describing dataset, cleaning rules, metrics, visualizations, and limitations.

---

## Notebook 1 — Ingestion (`part1_taxi/01_ingest.ipynb`)

**Goals**

- Programmatically download the required raw data files.
- Save them to the standardized project directory.
- Perform basic sanity checks on the loaded data.

**Tasks**

1. **Download raw files** using an appropriate Python package (e.g., `requests`, `urllib`, `wget`, or similar):  
   - `yellow_tripdata_2023-01.parquet`  
   - `taxi_zone_lookup.csv`
2. **Save files** to the following exact locations:  
   - `data/raw/tlc/yellow_tripdata_2023-01.parquet`  
   - `data/raw/lookup/taxi_zone_lookup.csv`
3. **Load datasets** (e.g., with `pandas` or `pyarrow` / `duckdb`) and print for each dataset:  
   - total **row count**,  
   - **column list**,  
   - **data types** per column,  
   - **number of nulls per column** (top 10 columns by null count is sufficient).

---

## Notebook 2 — Cleaning & Feature Engineering (`part1_taxi/02_clean_features.ipynb`)

**Goal**

Create a **curated Parquet dataset** saved to:

- `data/curated/part1_taxi_curated.parquet`

### Required Fields in Curated Dataset

- **Geography fields**
  - `PULocationID`, `DOLocationID`
  - `PU_borough`, `PU_zone`, `DO_borough`, `DO_zone` (joined from the taxi zone lookup)

- **Time fields**
  - `tpep_pickup_datetime`, `tpep_dropoff_datetime`
  - `pickup_date`
  - `pickup_hour` (0–23)
  - `weekday` (Mon–Sun)
  - `week_of_year` (1–53)

- **Core numeric fields**
  - `passenger_count`
  - `trip_distance`
  - `fare_amount`
  - `tip_amount`
  - `total_amount`
  - `payment_type`

- **Engineered fields**
  - `trip_duration_min` = (dropoff − pickup) in minutes
  - `speed_mph` = distance / duration in hours (handle zero/negative safely)
  - `fare_per_mile` = `fare_amount` / `trip_distance` (handle zero safely)
  - `tip_rate` = `tip_amount` / `total_amount` (handle zero safely)
  - `distance_bucket` with exactly these bins:  
    - \[0, 1), \[1, 3), \[3, 5), \[5, 10), \[10, +)

### Cleaning Rules (Apply in Code and Report Counts)

For each rule, you must **count and report** how many rows are removed, and produce a **before/after table**:

1. Remove rows where **pickup or dropoff timestamp is missing**.  
2. Remove rows where **`trip_duration_min` ≤ 0**.  
3. Remove rows where **`trip_distance` ≤ 0**.  
4. Remove rows where **`passenger_count` is missing or ≤ 0**.  
5. Remove rows where **`total_amount` ≤ 0**.  
6. Define and remove **duplicates** using this key:  
   - (`tpep_pickup_datetime`, `tpep_dropoff_datetime`, `PULocationID`, `DOLocationID`, `total_amount`).

The final curated dataset after applying all rules is written to  
`data/curated/part1_taxi_curated.parquet`.

---

## Notebook 3 — Stats, EDA & Visualization (`part1_taxi/03_stats_eda_viz.ipynb`)

All summary metrics must be computed using **Pandas or DuckDB** on the curated dataset.

### A. Required Groupby Metrics

1. **Daily trip counts** grouped by `PU_borough` and `pickup_date`.  
2. **Mean and median of `fare_amount` and `trip_distance`** grouped by `PU_borough` and `weekday`.  
3. For each `pickup_hour`, the **proportion of trips by `payment_type`**.  
4. **Mean `tip_rate`** grouped by `distance_bucket`.

### B. Required Window-Style Metrics

1. For each `PU_borough`, compute a **7-day rolling mean of daily trip counts** ordered by `pickup_date`.  
2. For each `PU_borough`, compute **day-over-day difference** and **day-over-day percent change** of daily trip counts.

### C. Required EDA Questions

Answer all questions quantitatively and support them with tables/plots where appropriate.

1. **Weekday effect by borough**  
   - For each borough, compute the weekday effect:
     \[
     S_{\text{borough}} =
     \frac{\max(\mu_{\text{weekday}, \text{borough}}) - \min(\mu_{\text{weekday}, \text{borough}})}{\mu_{\text{borough}}}
     \]
     where \(\mu_{\text{weekday}, \text{borough}}\) is the mean daily trips for that weekday in that borough, and \(\mu_{\text{borough}}\) is the mean daily trips for that borough overall.  
   - Identify **which borough has the strongest weekday effect** and report the value of \(S_{\text{borough}}\).

2. **Anomalous dates using z-scores**  
   - Compute a z-score on **daily total trip counts**.  
   - Identify **two specific dates** with anomalous total trip volume, show their **z-scores and dates**.

3. **Association of `tip_rate` with time vs distance**  
   - Compare how strongly `tip_rate` is associated with `pickup_hour` versus `distance_bucket`.  
   - Use a **quantitative comparison** (e.g., variance explained, correlation-like measures, or between-group variance) and state clearly **which factor is more strongly associated**, with justification.

### D. Required Visualizations

Include at least the following four plots:

1. **Time series of daily trips for each borough** with a **rolling 7-day average** overlaid.  
2. **Before/after cleaning distribution** of `trip_duration_min` (e.g., histograms or density plots).  
3. **Relationship plot** of `trip_distance` vs `fare_amount` (e.g., scatter plot or hexbin).  
4. **Heatmap** of **average trips by `weekday` (rows) and `pickup_hour` (columns)**.

---

## Report (`part1_taxi/report.pdf`)

The report should summarize the full analysis and results. It must include:

- **Dataset description**: grain (what a row represents), time range (January 2023), and key columns.  
- **Cleaning rules**: description of each rule and the **before/after removal table** showing how many rows were dropped at each step.  
- **Required metrics with interpretation**: explain what the groupby and window metrics reveal about demand, pricing, and temporal patterns.  
- **Required four plots with captions**: clearly label axes and explain what each plot shows.  
- **Limitations and future data needs**: discuss data quality issues, potential biases, and what additional data you would want next (e.g., weather, events, surge pricing, other months).

---

## Environment & Dependencies

The analysis is implemented in Python using standard data science libraries. See `requirements.txt` / `pyproject.toml` for the exact environment (e.g., `pandas`, `pyarrow`, `duckdb`, `matplotlib`, `seaborn`, `plotly`, etc.).  

Run notebooks in order:

1. `part1_taxi/01_ingest.ipynb`  
2. `part1_taxi/02_clean_features.ipynb`  
3. `part1_taxi/03_stats_eda_viz.ipynb`  

to reproduce the curated dataset, metrics, and figures used in `part1_taxi/report.pdf`.

