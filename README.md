# **Weather Analytics & Trends** 

Overview
---
* Setting up the environment
* Getting source data (live or near real-time train running status)
* A clear folder structure and commands
* Processing with PySpark (batch & streaming)
* Recommended Superset visualizations

# Weather Analytics Project — Setup, Data, Spark Processing & Superset Visualizations

*  complete step-by-step guide to set up a reproducible project that fetches weather data (OpenWeather API), processes it with Spark, stores results, and visualizes with Apache Superset.

## Architecture Overview

```plaintext
Input Data (API / File / Stream)
        │
        ▼
Read Source Using Spark
        │
        ▼
Write Raw Data into HDFS
        │
        ▼
Process Data in Spark
        │
        ▼
Write Processed Data into PostgreSQL
        │
        ▼
View on Superset Dashboard
```

---

## 1. Overview

This document covers:

- Creating a local/dev environment (Python, Docker) for the Weather Tracker
- Ingesting live and historical weather data from an API (e.g., OpenWeatherMap)
- Processing and enriching the data using PySpark (batch & streaming)
- Storing processed data in Parquet / HDFS / PostgreSQL
- Building interactive dashboards in Apache Superset

**Architecture Flow:**

```bash
OpenWeather API --> Raw JSON/CSV --> Spark Jobs --> Processed Parquet / DB --> Superset Dashboards
```

---

## 2. Prerequisites

- **Git**
- **Docker & Docker Compose**
- **Python 3.9+**
- **Java 11+**
- **Apache Spark 3.x**
- API Key from [OpenWeatherMap](https://openweathermap.org/api)

Optional: HDFS, Airflow for scheduling.

---

## 3. Repository Layout

```
weather-tracker/
│
├─ infra/                  # Docker, Spark, Superset configs
│   ├─ docker-compose.yml
│   ├─ spark/
│   └─ superset/
│
├─ data/                   # Raw & processed data
│   ├─ raw/
│   └─ processed/
│
├─ notebooks/              # Jupyter notebooks for exploration
│
├─ pyspark_jobs/           # ETL jobs
│   ├─ fetch_weather_data.py
│   ├─ process_weather_data.py
│   └─ utils/
│
├─ superset/               # Superset dashboard exports & configs
│
├─ scripts/                # Utility scripts for ingestion/testing
│
├─ dags/                   # Airflow DAGs for scheduled
````

---

## 4. Environment Setup

1. **Create virtual environment & install dependencies**
```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install requests pandas pyspark pyarrow fastparquet sqlalchemy psycopg2-binary
````

2. **Set API Key in environment variables**

```bash
export OPENWEATHER_API_KEY="your_api_key_here"
```

3. **Docker Compose (Spark, Superset, PostgreSQL)**

```bash
docker-compose up -d
```

---

## 5. Data Ingestion

### Example Python script to fetch weather data:

```python
import requests
import pandas as pd
from datetime import datetime
import os

API_KEY = os.getenv("OPENWEATHER_API_KEY")
CITY = "Mumbai"
URL = f"http://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}&units=metric"

response = requests.get(URL)
data = response.json()

df = pd.DataFrame([{
    "city": CITY,
    "timestamp": datetime.utcnow(),
    "temp": data["main"]["temp"],
    "humidity": data["main"]["humidity"],
    "pressure": data["main"]["pressure"],
    "wind_speed": data["wind"]["speed"],
    "weather": data["weather"][0]["description"]
}])

df.to_csv("data/raw/weather_data.csv", index=False)
```

---

## 6. Spark Processing

* **Batch Jobs** — Clean, format, and enrich daily weather data
* **Streaming Jobs** — Use Structured Streaming to continuously process incoming API data

Example Spark transformation:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, round

spark = SparkSession.builder.appName("WeatherProcessing").getOrCreate()

df = spark.read.csv("data/raw/weather_data.csv", header=True, inferSchema=True)

df_processed = df.withColumn("temp_celsius", round(col("temp"), 2)) \
                 .withColumn("humidity_percent", col("humidity").cast("int"))

df_processed.write.mode("overwrite").parquet("data/processed/weather_data.parquet")
```

---

## 7. Superset Visualizations

**Recommended Dashboards:**

1. **Temperature Trend Over Time** — Line chart showing temperature variation over a selected time range.
2. **Humidity & Pressure Trends** — Dual-axis chart comparing humidity and pressure changes.
3. **Weather Condition Frequency** — Pie or bar chart showing occurrences of different weather conditions (e.g., clear, rain, cloudy).
4. **Top 5 Hottest & Coldest Days** — Table highlighting dates with highest and lowest temperatures.
5. **Wind Speed Distribution** — Histogram of wind speeds across the dataset.
6. **Temperature vs Humidity Heatmap** — Color-coded chart showing correlation between temperature and humidity.
7. **City Comparisons** — Compare multiple cities’ temperatures in the same time frame.
8. **Anomaly Detection Chart** — Highlight extreme weather events (e.g., temp > 40°C or < 5°C).
9. **Rainfall Trends** — If precipitation data available, show rainfall accumulation over time.
10. **Day/Night Temperature Difference** — Bar chart showing the gap between daytime and nighttime temperatures.

---

## **1. Current Weather Overview**

* **KPI Cards**

  * Current Temperature (°C)
  * Current Humidity (%)
  * Wind Speed (km/h)
  * Precipitation probability (%)

---

## **2. Hourly / Daily Trends**

* **Line Chart**

  * Temperature over the last 24 hours
  * Humidity trends throughout the day
* **Area Chart**

  * Rainfall intensity forecast for the next 12 hours
  * Wind speed variation

---

## **3. Location-based Weather**

* **Map (Geo Chart)**

  * Current temperatures across different cities
  * Color-coded heatmap for temperature or rainfall
  * Wind direction overlay

---

## **4. Historical Weather Analysis**

* **Time Series Chart**

  * Average temperature trends for the past week/month
  * Rainfall vs. temperature comparison
* **Dual-axis Chart**

  * Temperature on one axis, humidity on another for correlation analysis

---

## **5. Forecast & Alerts**

* **Bullet Chart / Progress Bars**

  * Chance of rain vs. historical average for the same day
* **Custom Alert Table**

  * Show upcoming extreme weather alerts (e.g., heatwave, heavy rain)

---

## **6. Additional Creative Visuals**

* **Gauge Chart**

  * UV index
  * Air Quality Index (AQI) if API provides it
* **Calendar Heatmap**

  * Daily temperature highs/lows over the past year

---

## 8. Scheduling & Automation (Optional)

* Use **Airflow** or **cron jobs** to fetch data every 10-15 minutes
* Store raw data for historical analysis
* Refresh Superset dashboard automatically

---

## 9. Future Enhancements

* Integrate forecast API to predict future weather
* Combine with pollution & air quality index (AQI) data
* Deploy dashboards publicly

## 10. Tips & Considerations

* API limits: respect rate limits with retries and delays
* Timezones: normalize timestamps for consistency
* Data quality: handle missing or null values
* Dashboard speed: pre-aggregate heavy metrics for Superset

## Reference Links
* [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
* [Apache Superset Documentation](https://superset.apache.org/docs/intro)
* [Apache Airflow Documentation](https://airflow.apache.org/docs/)
* [HDFS Overview](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)
* [PostgreSQL Documentation](https://www.postgresql.org/docs/)
* [OpenWeatherMap API](https://openweathermap.org/api)
* [WeatherAPI](https://www.weatherapi.com/)
* [Open-Meteo](https://open-meteo.com/)
* [Google Maps Weather API Overview](https://developers.google.com/maps/documentation/weather/overview)
