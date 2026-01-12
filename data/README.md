# RoutePulse Dataset

## Download NYC Yellow Taxi Data

### Source
NYC Taxi & Limousine Commission
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

### Steps
1. Go to the link above
2. Download "Yellow Taxi Trip Records" for December 2024
3. Save the CSV file to this folder: `data/taxi.csv`

## Dataset Info

- **Records:** 3.5M+ taxi trips
- **Size:** ~1.2 GB
- **Key Columns:**
  - tpep_pickup_datetime
  - trip_distance
  - fare_amount
  - tip_amount
  - payment_type
  - passenger_count

## Load in Python
```python
import pandas as pd
df = pd.read_csv('data/taxi.csv')
print(df.shape)
```

## Load in Spark
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("RoutePulse").getOrCreate()
df = spark.read.csv("data/taxi.csv", header=True, inferSchema=True)
```
