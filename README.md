# IoT-Smart-Energy-Consumption-Analytics-Delta-Live-Tables-DLT-Pipeline
his project demonstrates how to design an end-to-end data pipeline using Databricks Delta Live Tables (DLT) to process IoT Smart Energy data from ingestion to analytics, following the Medallion Architecture.

## Project Overview
The pipeline processes smart meter data from IoT devices to monitor electricity usage, detect anomalies, and generate real-time analytical insights.
Landing (Azure Data Lake Gen2)
     ↓
Bronze (Raw Ingestion)
     ↓
Silver (Validation & Cleaning)
     ↓
Silver Quarantine (Anomalies)
     ↓
Gold (Aggregated Insights)

## Bronze Layer – Raw Data Ingestion
Source: JSON data from the landing folder

Ingestion: Auto Loader (cloud_files)

Captured Metadata:

_metadata.file_name

_metadata.file_modification_time

## Silver Layer – Validation & Business Rules
Data Validation Rules->
| Field          | Validation                           |
| -------------- | ------------------------------------ |
| timestamp      | must be valid datetime               |
| consumption_kw | numeric and non-null                 |
| voltage        | numeric or filled with default       |
| status         | must be from (ok, lowbattery, error) |
| temperature_c  | optional but numeric if present      |

Business Rules->
| Rule                        | Reason                    |
| --------------------------- | ------------------------- |
| consumption_kw >= 0         | Energy cannot be negative |
| voltage between 180–260     | Safe operating voltage    |
| timestamp in expected range | Reject faulty timestamps  |

Used APPLY CHANGES INTO for deduplication → iot_cleaned_silver_table.

## Silver Anomaly Quarantine
| Condition                             | Category                       |
| ------------------------------------- | ------------------------------ |
| Voltage < 180V AND Consumption > 8 kW | Tampering / bypass             |
| Consumption = 0 AND Status = OK       | Theft / meter stuck            |
| Voltage > 260V                        | Hardware error / line issue    |
| High load + LOWBATTERY                | Device may stop reporting soon |
Table Name → silver_anomaly_quarantine

## Gold Layer – Analytical Views

| Table                    | Logic                                 |
| ------------------------ | ------------------------------------- |
| `city_hourly_usage`      | SUM(consumption_kw) by city & hour    |
| `daily_meter_usage`      | SUM(consumption_kw) per meter per day |
| `voltage_anomaly_alerts` | voltage < 180 or > 260                |
| `top_consuming_meters`   | Top 10 meters by daily consumption    |

## Tech Stack

Databricks Delta Live Tables (DLT)

Azure Data Lake Gen2

PySpark / SQL

JSON Data Source

Medallion Architecture
