# 🚴 GeoBike-Station-Planner

**GeoBike-Station-Planner** is a Python-based tool leveraging geospatial analysis to optimize new bike station placements in Washington, D.C. The project employs techniques like **geofencing**, **grid-based clustering**, and **service radius checks** to identify high-demand, underserved areas, providing actionable insights for expanding the bike-sharing network.

## 📍 Project Overview

To help Capital Bikeshare determine the most strategic locations for new bike stations, I conducted a thorough geospatial analysis using recent trip data. The main objective was to **pinpoint high-demand areas** that are currently underserved by existing bike stations. This project aims to **enhance service coverage** and improve overall rider satisfaction by suggesting optimal locations for future station placements.

## 🔍 Features

- **Geospatial Analysis**: Uses trip data to analyze bike usage patterns and station coverage.
- **Grid-based Clustering**: Identifies clusters of high-demand areas that are underserved.
- **Geofencing**: Defines service areas and checks coverage within predefined zones.
- **Service Radius Check**: Ensures proposed locations fill gaps in the current network.
- **Cloud-based Processing**: Utilizes Google Colab for scalable and efficient data analysis.

## 📊 Datasets

The analysis is based on **Capital Bikeshare's public trip data** for the last three months. The dataset includes:

- **Trip Start and End Times**
- **Trip Durations**
- **Station Locations (Latitude, Longitude)**

You can access the dataset [here](https://capitalbikeshare.com/system-data).

## 🛠️ Data Loading and Cleaning

The analysis was performed in a **Google Colab** environment for its powerful cloud-based capabilities. Data was loaded into Pandas DataFrames from CSV files for each month, merged, and cleaned to ensure consistency. The following steps outline the process:

```python
import pandas as pd

# Load the CSV files
df_aug = pd.read_csv('/content/202408-capitalbikeshare-tripdata.csv')
df_sep = pd.read_csv('/content/202409-capitalbikeshare-tripdata.csv')
df_oct = pd.read_csv('/content/202410-capitalbikeshare-tripdata.csv')

# Merge the DataFrames
df = pd.concat([df_aug, df_sep, df_oct], ignore_index=True)

# Filter and rename columns for consistency
start_stations = df[['start_station_name', 'start_lat', 'start_lng']].dropna().rename(
    columns={'start_station_name': 'station_name', 'start_lat': 'latitude', 'start_lng': 'longitude'}
)
end_stations = df[['end_station_name', 'end_lat', 'end_lng']].dropna().rename(
    columns={'end_station_name': 'station_name', 'end_lat': 'latitude', 'end_lng': 'longitude'}
)
