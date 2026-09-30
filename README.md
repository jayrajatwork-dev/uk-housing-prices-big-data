# NRW Rail Delay Analytics

A cloud-based analytics platform that turns real-time Deutsche Bahn data into delay insights and delay-risk predictions for rail travel across North Rhine-Westphalia, Germany.

**Status: Complete.** Built as a solo M.Sc. Applied IT Project (Mediadesign Hochschule, Düsseldorf) between July and September 2026. The pipeline collected live data from 24 July to 29 September 2026.

---

## The problem

Rail delay information in NRW is reactive: travellers only learn a train is late once it already is. There is no accessible way to see which lines and stations are structurally unreliable, or to estimate delay risk before a departure.

## The solution

An end-to-end cloud pipeline that:

- **Ingests** live delay data every 5 minutes from 20 major NRW stations via the Deutsche Bahn Timetables API
- **Enriches** it with hourly weather data from Open-Meteo
- **Stores** it in a PostgreSQL data warehouse, deduplicated and cleaned
- **Predicts** delay risk with a gradient boosting model
- **Visualizes** delay patterns in an interactive Power BI dashboard

---

## Dashboard

![Dashboard overview](docs/screenshots/dashboard-overview.png)
*Station map with headline figures and delay by hour of day.*

![Category and station breakdown](docs/screenshots/dashboard-breakdown.png)
*Delay distribution by train category and station.*

---

## Tech stack

| Layer | Technology |
|---|---|
| **Compute** | Azure Functions (Python 3.13, Flex Consumption plan, timer-triggered) |
| **Storage** | Azure Data Lake Storage Gen2 |
| **Database** | Azure Database for PostgreSQL, Flexible Server |
| **Streaming (prototype)** | Azure Event Hubs with the Kafka protocol |
| **Machine learning** | scikit-learn (HistGradientBoosting), SHAP for explainability |
| **Visualization** | Power BI Desktop |
| **Data sources** | Deutsche Bahn Timetables and Station Data APIs, Open-Meteo |

All infrastructure ran on **Microsoft Azure** (Switzerland North) under an Azure for Students subscription.

---

## Architecture

```
DB Timetables API
      |
      v
Azure Function (polls every 5 min)
      |
      v
Data Lake Gen2 (raw JSON, by station / date / hour)
      |
      v
Scheduled Azure Function (incremental transform + weather refresh)
      |
      v
PostgreSQL warehouse  ----->  Power BI dashboard
      |
      +---------------------->  Delay-risk model (scikit-learn)
```

Both functions run on a schedule in Azure, so the system keeps itself up to date without any local machine.

---

## Results

Over the collection period the warehouse stored more than 100,000 stop events from 20 stations. Replacement-bus services were excluded from analysis, since they are added during disruptions and have no original timetable to be late against.

The model was trained on the earlier weeks and tested on the most recent two weeks of data, which it had never seen.

| Model | Result | Naive baseline |
|---|---|---|
| Delay-risk classifier (delayed more than 5 min, yes or no) | **ROC-AUC 0.781** | 0.500 |
| Delay-minutes regression | R² -0.019 | 0 (predicting the mean) |
| Delay-range classifier (4 ranges) | Accuracy 39.7% | 65.5% (always the most common range) |

**How to read these numbers.** The source feed only reports stops where something changed, so about 93% of the test records are delayed. On data this lopsided, a model that always says "delayed" already scores about 93% accuracy, so accuracy is not a meaningful measure. ROC-AUC is: the classifier ranks a delayed stop above an on-time one about 78% of the time, against 50% by chance.

Predicting *how many minutes* a train will be late did not beat simple baselines. Station, train type, time and weather indicate whether a delay is likely, but its length appears to depend on incident-specific factors that are not in this data.

According to SHAP, the strongest drivers of delay risk were train category, hour of day and temperature. Precipitation and wind contributed comparatively little.

---

## Key findings

- **Detected a real infrastructure incident independently.** Delays of up to 1,388 minutes and unexpected negative delays were traced to a construction-train derailment that closed the Aachen-Köln line in late July 2026, confirmed against public news reports. The raw data was kept, and outliers above 180 minutes are excluded in a separate, documented view.
- **Diagnosed source-API behaviour empirically.** The DB `/fchg` endpoint returns full current-state snapshots on every poll, not incremental changes. The first load stored 1.16 million rows; redesigning it as an upsert reduced this to the genuine unique events.
- **Tested assumptions against real data.** An API field that looked like a cancellation flag was tested on 500 real files and rejected, which led to the correct field for identifying added bus services.
- **Reported limitations honestly.** Delay-magnitude prediction was attempted with two different approaches, and both failed in the same way. This is documented as a finding.

---

## Repository structure

```
├── ingestion/          Azure Function that polls the DB API and writes to Data Lake
├── pipeline/           Transform, weather backfill, scheduled automation
├── ml/                 Model training notebook and scoring scripts
├── sql/                Database schema and views
├── kafka-experiment/   Standalone Kafka / Event Hubs producer and consumer
└── docs/               Dashboard screenshots
```

---

## Setup

1. Clone the repository and copy `.env.example` to `.env`, filling in your own credentials (DB API, Azure Storage, PostgreSQL).
2. `pip install -r ingestion/requirements.txt`
3. Deploy `ingestion/` as an Azure Function App (Python 3.13, Flex Consumption) and add the same values as app settings.
4. Run `sql/schema.sql` against your PostgreSQL instance.
5. Run `pipeline/transform.py` and `pipeline/weather_backfill.py` once to backfill existing data. After that, the scheduled function keeps the warehouse current.
6. Open `ml/NRW_Delay_Model_Training.ipynb`. It reads directly from the `ml_training_data` view, so no export step is needed. (`ml/explore_data.py` is optional, for quick data exploration.)
7. Connect Power BI Desktop to your PostgreSQL instance to rebuild the dashboard.

---

## Possible extensions

- Live predictions for upcoming departures, using weather forecasts instead of observed weather
- More stations and regions beyond NRW
- A simple user-facing interface for the delay predictor
- Features that describe incidents (for example disruption messages), which may make delay-length prediction possible

---

## Author

Jayraj Maghade, M.Sc. Information Technology (AI and Data Analytics), Mediadesign Hochschule Düsseldorf

[LinkedIn](https://www.linkedin.com/in/jayraj-maghade/)
