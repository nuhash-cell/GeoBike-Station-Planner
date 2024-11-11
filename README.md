# 🚴 GeoBike-Station-Planner

**GeoBike-Station-Planner**: A Python-based tool leveraging geospatial analysis to optimize new bike station placements in Washington, D.C. Features include **geofencing**, **grid-based clustering**, and **service radius checks** to identify high-demand, underserved areas.

## 🗺️ Project Overview

Help to determine the next place to build a bike station.  
To help **Capital Bikeshare** find the best spots for new bike stations, I used trip data and geospatial analysis. The goal was simple: pinpoint high-demand areas that current stations don’t cover well. The results show exactly where new stations would get the most use, improving service for riders.

## 📊 Datasets

I used trip data from Capital Bikeshare’s public system, focusing on the **last 3 months**. The dataset, available [here](https://capitalbikeshare.com/system-data), includes:

- **Trip Start and End Times**
- **Trip Durations**
- **Station Locations**

This dataset offers a complete view of recent bike usage in Washington, D.C.

## 🛠️ Data Loading and Cleaning

I used **Google Colab** to run the analysis, leveraging its cloud-based environment for efficient data handling. The data was loaded into **Pandas DataFrames** from CSV files for each month. I merged the datasets into a single DataFrame and cleaned the data by removing rows with missing station names. Column names were standardized for consistency.

### 🔄 Code

```python
import pandas as pd

# Load the CSV files
df_aug = pd.read_csv('/content/202408-capitalbikeshare-tripdata.csv')
df_sep = pd.read_csv('/content/202409-capitalbikeshare-tripdata.csv')
df_oct = pd.read_csv('/content/202410-capitalbikeshare-tripdata.csv')

# Merge the DataFrames
df = pd.concat([df_aug, df_sep, df_oct], ignore_index=True)

# Filter out rows where start_station_name or end_station_name is not NaN
start_stations = df[['start_station_name', 'start_lat', 'start_lng']].dropna()
end_stations = df[['end_station_name', 'end_lat', 'end_lng']].dropna()

# Rename columns to make them consistent for combining
start_stations.rename(columns={'start_station_name': 'station_name',
                               'start_lat': 'latitude',
                               'start_lng': 'longitude'}, inplace=True)

end_stations.rename(columns={'end_station_name': 'station_name',
                             'end_lat': 'latitude',
                             'end_lng': 'longitude'}, inplace=True)

