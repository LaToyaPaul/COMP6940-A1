# COMP6940 — Big Data and Data Visualization

## Assignment 1 · Part 1: NYC Yellow Taxi — Demand, Pricing, and Temporal Patterns

> **The University of the West Indies, St. Augustine**  
> Department of Computing and Information Technology · January 2026

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Notebooks](#running-the-notebooks)
- [Notebooks](#notebooks)
- [Outputs](#outputs)
- [Authors](#authors)

---

## Overview

This project analyses NYC Yellow Taxi trip records for January 2023. The analysis covers data ingestion, cleaning and feature engineering, non-trivial statistical metrics (groupby and window-style), and exploratory data analysis with visualizations.

The work is structured across three Jupyter notebooks following the required repository layout for COMP6940 Assignment 1 (Part 1).

---

## Dataset


| Dataset                                 | Source                                                                                                  | Format  |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------- |
| NYC Yellow Taxi Trip Records (Jan 2023) | [TLC Trip Record Data](https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet) | Parquet |
| Taxi Zone Lookup Table                  | [TLC Misc](https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv)                             | CSV     |


Raw data is downloaded programmatically in `01_ingest.ipynb` and saved to `data/raw/`. Do **not** commit raw data files to version control.

---

## Repository Structure

```
COMP6940-A1/
├── README.md
├── requirements.txt
├── data/
│   ├── raw/
│   │   ├── tlc/
│   │   │   └── yellow_tripdata_2023-01.parquet
│   │   └── lookup/
│   │       └── taxi_zone_lookup.csv
│   └── curated/
│       └── part1_taxi_curated.parquet
└── part1_taxi/
    ├── 01_ingest.ipynb
    ├── 02_clean_features.ipynb
    ├── 03_stats_eda_viz.ipynb
    ├── report.pdf
    └── data_dictionary.pdf
```

---

## Getting Started

### Prerequisites

- Python 3.9+ (we used 3.12)
- Jupyter Notebook or alternative notebook environment

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/aaanthonyyy/COMP6940-A1.git && cd COMP6940-A1
```

1. **Create and activate a virtual environment**

Using `uv` (recommended):

```bash
uv venv .venv
source .venv/bin/activate
```

Or using `pip`:

```bash
python -m venv .venv
source .venv/bin/activate
```

1. **Install dependencies**

Using `uv` (recommended):

> ENsure you have uv.lock in your repository.

```bash
uv sync
```

Using `pip`:

```bash
pip install -r requirements.txt
```

### Running the Notebooks

> [!IMPORTANT]
> Notebooks must be run in order. Each notebook depends on the outputs of the previous one.

Then open and run each notebook in sequence:

```
01_ingest.ipynb          ← downloads and saves raw data
02_clean_features.ipynb  ← cleans data and engineers features
03_stats_eda_viz.ipynb   ← statistics, EDA, and visualizations
```

---

## Notebooks


| Notebook                  | Description                                                                                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `01_ingest.ipynb`         | Downloads raw Parquet and CSV files via `requests`, saves to `data/raw/`, prints row counts, column lists, data types, and null counts                        |
| `02_clean_features.ipynb` | Applies 6 cleaning rules with a before/after removal table, engineers all required fields, saves curated dataset to `data/curated/part1_taxi_curated.parquet` |
| `03_stats_eda_viz.ipynb`  | Computes all groupby and window metrics using Pandas, answers 3 EDA questions with quantitative evidence, produces exactly 4 required plots                   |
