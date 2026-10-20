# Uber Data Analytics | Data Engineering Project I

The goal of this project is to perform data analytics on Uber (NYC TLC taxi trip) data using Python, Jupyter, the Mage data pipeline tool, Microsoft SQL Server and Power BI. A local CSV file is extracted, transformed into a star schema (one fact table and seven dimension tables) with pandas, loaded into SQL Server by a Mage pipeline, and visualized in Power BI.

## Architecture / Flow

![Process Flow](Process%20Flow.png)

```
Local CSV (uber_data.csv) -> Mage (extract -> transform -> load) -> MS SQL Server -> Power BI
```

## Data Model

![Uber Data Flow Diagram](Uber%20Data%20Flow%20Diagram.png)

- Fact table: `fact_table` (VendorID, foreign keys to each dimension, store_and_fwd_flag, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount)
- Dimensions: `datetime_dim`, `passenger_count_dim`, `trip_distance_dim`, `rate_code_dim`, `pickup_location_dim`, `dropoff_location_dim`, `payment_type_dim`

## Technology Used

- Programming language: Python (pandas)
- Jupyter Notebook
- Mage AI - modern data pipeline tool: https://www.mage.ai/
- Microsoft SQL Server
- Power BI

## Repository Structure

```
.
├── Uber Data Engineering Project.ipynb   # Exploratory star-schema modelling in pandas
├── 1.extract.py                          # Mage data loader block
├── 2.transform.py                        # Mage transformer block (builds fact + dimensions)
├── 3.load.py                             # Mage data exporter block (writes to MSSQL)
├── uber_data.csv                         # Source dataset
├── Process Flow.png                      # Pipeline flow diagram
├── Uber Data Flow Diagram.png            # Star schema diagram
├── README.md
└── README.txt                            # Earlier version of the project notes
```

## Pipeline Blocks

**1. Extract (`1.extract.py`)**
- Mage `@data_loader` that loads `uber_data.csv` from a local Windows path with `FileIO().load()`, with a test asserting the output is not empty.

**2. Transform (`2.transform.py`)**
- Converts `tpep_pickup_datetime` and `tpep_dropoff_datetime` to datetimes.
- Builds `datetime_dim` with hour, day, month, year and weekday for pickup and drop-off.
- Builds de-duplicated dimensions with surrogate IDs for passenger count, trip distance, pickup and drop-off coordinates.
- Maps `RatecodeID` to names (Standard rate, JFK, Newark, Nassau or Westchester, Negotiated fare, Group ride) and `payment_type` to names (Credit card, Cash, No charge, Dispute, Unknown, Voided trip).
- Joins the dimensions back to the trips to produce `fact_table`, and returns all tables as dictionaries.

**3. Load (`3.load.py`)**
- Mage `@data_exporter` that writes each returned table to SQL Server schema `dbo` (table name = dictionary key, `if_exists='replace'`), using the `default` profile in Mage's `io_config.yaml`.

**Notebook (`Uber Data Engineering Project.ipynb`)**
- Exploratory pandas version of the same modelling: reads `uber_data.csv`, removes duplicate rows, adds a `trip_id`, and builds the dimensions and `fact_table`.

## Dataset Used

TLC Trip Record Data. Yellow and green taxi trip records include fields capturing pick-up and drop-off dates/times, pick-up and drop-off locations, trip distances, itemized fares, rate types, payment types, and driver-reported passenger counts.

`uber_data.csv` columns: VendorID, tpep_pickup_datetime, tpep_dropoff_datetime, passenger_count, trip_distance, pickup_longitude, pickup_latitude, RatecodeID, store_and_fwd_flag, dropoff_longitude, dropoff_latitude, payment_type, fare_amount, extra, mta_tax, tip_amount, tolls_amount, improvement_surcharge, total_amount.

Dataset used in the video: https://github.com/darshilparmar/uber-etl-pipeline-data-engineering-project/blob/main/data/uber_data.csv

More info about the dataset:
1. Website - https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
2. Data Dictionary - https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf

## Process

1. Store the data on a local drive.
2. Load the data into Mage AI:
   - Data extraction
   - Data transformation
   - Data loading
   - Adding triggers
3. Store the data in MS SQL Server.
4. Visualize the data in Power BI.

## How to Run

1. Install Mage (`pip install mage-ai`) and start a project (`mage start <project_name>`).
2. Create a pipeline with three blocks using `1.extract.py`, `2.transform.py` and `3.load.py`, in that order.
3. Update the file path in `1.extract.py` to point at your copy of `uber_data.csv`.
4. Add SQL Server connection settings to the `default` profile in the project's `io_config.yaml` (keep credentials out of version control).
5. Run the pipeline (optionally add a trigger), then connect Power BI to the `dbo` tables.
