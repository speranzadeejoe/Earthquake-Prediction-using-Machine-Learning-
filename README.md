# Earthquake-Prediction-using-Machine-Learning-
# Earthquake Data Visualization using PySpark, MongoDB, and Bokeh

## Overview
This project visualizes earthquake data using PySpark for data processing, MongoDB for data storage, and Bokeh for interactive visualizations. The visualization includes:
1. A world map with earthquake locations.
2. A bar chart for the frequency of earthquakes by year.
3. A line chart for maximum and average earthquake magnitudes by year.

## Prerequisites
Ensure you have the following installed:
- Python (>=3.7)
- Apache Spark (PySpark)
- MongoDB
- Bokeh
- Pandas
- GeoPandas
- PyMongo

## Setup

### 1. Install Dependencies
```sh
pip install pyspark pymongo bokeh pandas geopandas
```

### 2. Start MongoDB
Ensure MongoDB is running locally or on a remote server.
```sh
mongod --dbpath <your-db-path>
```

### 3. Load Earthquake Data into MongoDB
```python
from pymongo import MongoClient
import pandas as pd

client = MongoClient("mongodb://localhost:27017/")
db = client["earthquakeDB"]
collection = db["earthquakes"]

data = pd.read_csv("earthquake_data.csv")
records = data.to_dict(orient='records')
collection.insert_many(records)
```

### 4. Process Data Using PySpark
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import year, avg, max

spark = SparkSession.builder.appName("EarthquakeAnalysis").getOrCreate()

# Load data from MongoDB
df = spark.read.format("mongo").option("uri", "mongodb://localhost:27017/earthquakeDB.earthquakes").load()

df = df.select("time", "latitude", "longitude", "mag")
df = df.withColumn("year", year(df["time"]))

# Aggregation
yearly_data = df.groupBy("year").agg(max("mag").alias("max_mag"), avg("mag").alias("avg_mag"))
frequency_data = df.groupBy("year").count()
```

### 5. Create Visualizations with Bokeh
```python
from bokeh.plotting import figure, show, output_file
from bokeh.tile_providers import get_provider, Vendors
from bokeh.models import ColumnDataSource

# Map Visualization
earthquakes = df.toPandas()
source = ColumnDataSource(earthquakes)

p = figure(title="Earthquake Map", x_axis_type="mercator", y_axis_type="mercator")
p.add_tile(get_provider(Vendors.CARTODBPOSITRON))
p.circle(x="longitude", y="latitude", source=source, size=8, color="red", alpha=0.6)

# Frequency Visualization
freq = frequency_data.toPandas()
p1 = figure(title="Frequency of Earthquakes by Year", x_axis_label="Years", y_axis_label="Number of Occurrences")
p1.vbar(x=freq["year"], top=freq["count"], width=0.5, color="red")

# Magnitude Visualization
mag_data = yearly_data.toPandas()
p2 = figure(title="Maximum and Average Magnitude by Year", x_axis_label="Years", y_axis_label="Magnitude")
p2.line(mag_data["year"], mag_data["max_mag"], color="red", legend_label="Max Magnitude")
p2.line(mag_data["year"], mag_data["avg_mag"], color="yellow", legend_label="Avg Magnitude")

# Output
output_file("earthquake_visualization.html")
show(p)
show(p1)
show(p2)
```

## Conclusion
This project integrates PySpark, MongoDB, and Bokeh to analyze and visualize earthquake data effectively. The combination of big data processing and interactive visualization helps in understanding earthquake trends and patterns efficiently.

