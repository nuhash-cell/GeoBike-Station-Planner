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
```


# 🚲 Extracting Existing Bike Stations

In the previous step, I filtered the dataset to include only trips where either `start_station_name` or `end_station_name` had valid entries, ensuring that only recognized bike stations were considered. The goal here was to compile a comprehensive list of these existing stations for further analysis.

## 🔍 Process Overview

1. **Combining Start and End Station Data**:  
   To capture all potential bike stations, I merged the filtered DataFrames for start and end stations. This ensured that stations listed as either a starting or ending location were included in the analysis.

2. **Removing Duplicates**:  
   Since the same station could appear multiple times (e.g., as both a start and end location), I used `drop_duplicates()` to retain only unique station names, along with their respective coordinates.

3. **Exporting the Station List**:  
   The final list of unique bike stations was saved to a CSV file for use in subsequent geospatial analysis.

## 🧑‍💻 Code

```python
import pandas as pd

# 🗂️ Combine start and end station data
all_stations = pd.concat([start_stations, end_stations])

# 🗃️ Remove duplicates to get unique station names
existing_stations = all_stations.drop_duplicates(subset=['station_name']).reset_index(drop=True)

# 📁 Save the list of existing stations to a CSV file
output_file_path = '/content/Existing_stations.csv'
existing_stations.to_csv(output_file_path, index=False)
```

# 🗺️ Geospatial Data Extraction and Geofencing

This step focuses on geospatial analysis using several Python libraries to handle data and perform calculations:

- **NumPy**: For numerical operations
- **GeoPandas**: For geospatial data manipulation
- **Shapely**: For geometric operations (e.g., convex hull)
- **OSMNx**: For road network extraction from OpenStreetMap
- **NetworkX**: For graph-based operations on the road network
- **Geopy**: For distance calculations using the geodesic formula

## 📋 Process Overview

1. **Extract Trip Coordinates**:  
   I combined the latitude and longitude of both starting and ending points from the trip data, keeping only entries with valid coordinates.

2. **Download Road Network and Extract Nodes**:  
   Using OSMNx, I downloaded the road network for Washington, D.C., focusing on bike paths, and extracted nodes representing intersections and road endpoints.

3. **Create a Geofence Using Convex Hull**:  
   I generated a geofence using the convex hull of the road network nodes to outline the area of interest, ensuring focus on the relevant urban area.

## 🧑‍💻 Code

```python
import numpy as np
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point
import osmnx as ox
import networkx as nx
from geopy.distance import geodesic

# 🗺️ Step 1: Extract Trip Coordinates
coordinates = pd.concat([
    df[['start_lat', 'start_lng']].rename(columns={'start_lat': 'latitude', 'start_lng': 'longitude'}),
    df[['end_lat', 'end_lng']].rename(columns={'end_lat': 'latitude', 'end_lng': 'longitude'})
]).dropna()
print(f"Loaded {len(coordinates)} coordinates.")

# 🏙️ Step 2: Download Road Network and Extract Nodes
place_name = "Washington, D.C., USA"
G = ox.graph_from_place(place_name, network_type="bike", simplify=True, retain_all=False)

# Extract nodes with longitude (x) and latitude (y)
nodes = ox.graph_to_gdfs(G, nodes=True, edges=False)
nodes_df = nodes[['x', 'y']].reset_index()

# 🛑 Step 3: Create a Geofence Using Convex Hull
nodes_gdf = gpd.GeoDataFrame(nodes_df, geometry=gpd.points_from_xy(nodes_df['x'], nodes_df['y']), crs="EPSG:4326")
geofence_polygon = nodes_gdf.unary_union.convex_hull

```
# 🗺️ Filtering Coordinates and Stations Within the Geofence

The dataset includes several trip points that extend beyond the main Washington, D.C. area. To focus on the urban core, I filtered both the trip coordinates and station data, including only points within a defined geofence around the city. This step ensures the analysis is relevant, avoiding noise from areas outside the city boundary.

## 📋 Process Overview

1. **Filtering Trip Coordinates**:  
   I converted the combined trip coordinates (start and end points) into a GeoDataFrame and applied the geofence filter.

2. **Filtering Existing Bike Stations**:  
   I processed the existing station data by converting it to a GeoDataFrame and applied the same geofence filter to retain only stations within the city boundary.

## 🧑‍💻 Code

```python
import geopandas as gpd

# 🗺️ Step 1: Convert Trip Coordinates to a GeoDataFrame
coordinates_gdf = gpd.GeoDataFrame(
    coordinates,
    geometry=gpd.points_from_xy(coordinates['longitude'], coordinates['latitude']),
    crs="EPSG:4326"
)

# 🌐 Filter trip coordinates that fall within the geofence polygon
filtered_coordinates = coordinates_gdf[coordinates_gdf.within(geofence_polygon)].reset_index(drop=True)

# 📁 Save the filtered trip coordinates to a CSV file
filtered_file_path = '/content/filtered_coordinates_within_geofence.csv'
filtered_coordinates.to_csv(filtered_file_path, index=False)

# 🏢 Step 2: Convert Existing Stations to a GeoDataFrame
stations_gdf = gpd.GeoDataFrame(
    Existing_stations,
    geometry=gpd.points_from_xy(Existing_stations['longitude'], Existing_stations['latitude']),
    crs="EPSG:4326"
)

# 🌐 Filter existing stations that fall within the geofence polygon
filtered_stations = stations_gdf[stations_gdf.within(geofence_polygon)].reset_index(drop=True)

# 📁 Save the filtered list of unique stations to a CSV file
filtered_output_path = '/content/filtered_unique_stations_within_geofence.csv'
filtered_stations[['station_name', 'latitude', 'longitude']].to_csv(filtered_output_path, index=False)
```
# 🗺️ Grid-Based Clustering Algorithm

Grid-based clustering is a method that divides a geographical area into uniform grid cells and assigns data points based on their coordinates. In this project, I used a grid size of **0.001 degrees** (approximately 100 meters) to segment Washington, D.C. into manageable sections. Each trip coordinate was mapped to a grid cell using its latitude and longitude. By aggregating data within each cell, I calculated the centroid (average position) and density (number of points) for each grid cell. This method efficiently identifies areas of high bike usage by focusing on the density of points in each grid cell.

## 📋 Process Overview

1. **Define Grid Size**:  
   The grid size was set to 0.001 degrees, roughly corresponding to a 100-meter cell in the latitude and longitude coordinates.

2. **Assign Trip Coordinates to Grid Cells**:  
   The coordinates were mapped to grid cells by calculating the indices based on their latitude and longitude.

3. **Calculate Centroid and Density**:  
   For each grid cell, the centroid (mean position) and density (count of points) were computed.

4. **Export Clustered Data**:  
   The resulting clustered data was saved to a CSV file for further analysis.

## 🧑‍💻 Code

```python
import pandas as pd

# 📏 Step 1: Define grid size (approx. 100 meters)
grid_size = 0.001

# 🗺️ Step 2: Calculate grid cell indices for longitude and latitude
coordinates_df['x_cell'] = (coordinates_df['longitude'] // grid_size).astype(int)
coordinates_df['y_cell'] = (coordinates_df['latitude'] // grid_size).astype(int)

# 📊 Step 3: Group by grid cell and compute centroid (mean) and density (count)
cluster_summary = coordinates_df.groupby(['x_cell', 'y_cell']).agg(
    x=('longitude', 'mean'),
    y=('latitude', 'mean'),
    density=('latitude', 'size')
).reset_index()

# 📁 Step 4: Save the clustered data to a CSV file
output_path = '/content/grid_clustered_coordinates.csv'
cluster_summary[['x', 'y', 'density']].to_csv(output_path, index=False)
```
