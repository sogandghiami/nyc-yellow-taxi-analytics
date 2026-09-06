# NYC Yellow Taxi — Driver & Fleet Profitability Analytics

> 🚧 **Status:** Core analysis (data cleaning, EDA, SQL modeling, ML) is complete. Power BI dashboard is in progress.

An end-to-end data analytics project that turns raw NYC taxi trip records into actionable insights for **drivers** and **ride-hailing / taxi companies** — answering the question: *"Where and when should I drive (or place my fleet) to maximize earnings?"*

---

## Motivation

Most public taxi-data projects stop at generic exploratory charts. This project was built around a concrete business use case instead:

- **For individual drivers:** identify which boroughs and hours of the day yield the best return per minute of driving — not just the highest average fare, but earnings *relative to trip duration*, so drivers can plan their working hours and areas more efficiently and estimate their expected income.
- **For taxi / ride-hailing companies (e.g. Uber-style fleets):** identify the time windows and zones where demand is highest, to help decide where to allocate more vehicles. (Note: the dataset only contains completed trips, not unmet demand or fleet size — see *Limitations* below.)

## Dataset

- **Source:** [NYC Taxi & Limousine Commission (TLC) Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) — official public dataset.
- **Data used for analysis:** Yellow Taxi trip records, January 2026.
- **Reproducibility:** the notebook includes a parameterized download function — any user can request a different month/year and the corresponding file is fetched automatically, instead of the pipeline being hardcoded to one static file.

```python
def download_taxi_data(year: int, month: int) -> str:
    """Downloads the NYC TLC Yellow Taxi parquet file for a given year/month
    and returns the local file path."""
    url = f"https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_{year}-{month:02d}.parquet"
    ...
```

## Tech Stack

| Layer | Tools |
|---|---|
| Data cleaning & EDA | Python, pandas, matplotlib |
| Data modeling | SQLite (star schema: fact + dimension tables) |
| Analysis | SQL (JOINs, GROUP BY, aggregate queries) |
| Machine Learning | scikit-learn (Linear Regression, Random Forest) |
| Dashboard | Power BI *(in progress)* |

## Pipeline / Architecture

```
Raw parquet (TLC)
   │
   ▼
Data Cleaning (pandas)          → duplicate removal, invalid RatecodeID fix,
   │                              datetime feature extraction
   ▼
Exploratory Data Analysis        → fare by hour/day/distance/rate type
   │
   ▼
Star Schema Modeling (SQLite)   → fact_trips + dim_location, dim_datetime,
   │                              dim_ratecode, dim_payment
   ▼
SQL Analysis                     → borough × hour earnings-per-minute analysis
   │
   ▼
ML Model (Random Forest)         → fare_amount prediction, R² = 0.83
   │
   ▼
Power BI Dashboard *(in progress)*
```

## Key Findings

- **Fare peaks between 5–7 AM**, likely driven by airport trips (long, fixed-rate rides) rather than traffic.
- **Trip volume is highest Thu–Sat**, but average fare stays flat across the week — the weekend surge is about *more riders*, not pricier trips.
- **Distance and fare are strongly linearly related** as expected, confirmed across both pandas and SQL analysis.
- **Data quality issue found and fixed:** ~4% of trips had `RatecodeID = 99` (officially "Null/unknown" per TLC docs). A naive groupby-mode fill failed silently when 99 was itself the majority value in a location group; fixed by excluding 99 before computing the mode.
- **Driver profitability ≠ trip volume:** Queens (largely due to JFK) has the highest earnings-per-minute, while Manhattan — despite having by far the most trips — ranks lower, since its trips are shorter and more traffic-heavy.
- **ML model:** Random Forest predicts `fare_amount` with **R² = 0.83, RMSE ≈ 7.0**, with `trip_distance` as by far the strongest predictor.

## Repository Structure

```
├── notebooks/
│   └── nyc_yellow_taxi_analysis.ipynb   # full pipeline: cleaning → EDA → SQL → ML
├── output/
│   └── uber_trips.db                     # SQLite database (star schema)
├── README.md
```

## How to Run

## How to Run

1. Clone the repository.
2. Run `download_taxi_data(year, month)` to fetch the dataset for any month you want to analyze — no manual setup, paths, or credentials required.
3. Run the notebook top to bottom.
> Note: `output/uber_taxi.db` and the trained model (`.joblib`) are not included in this repo (large binary files). Run the notebook end-to-end to regenerate them locally.

## Limitations

- The dataset only reflects **completed trips**, so it cannot directly measure unmet demand or how many taxis are actively working — only observed demand.
- `EWR` (Newark) and some `Staten Island` hourly bins were excluded/limited in the borough × hour analysis due to insufficient sample size (fewer than 20 trips), to avoid unreliable averages.
- Analysis is based on a single month of historical data; seasonal patterns are not captured.

## Author

Sogand Ghiami
