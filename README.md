# GeoBike-Station-Planner
GeoBike-Station-Planner: A Python-based tool leveraging geospatial analysis to optimize new bike station placements in Washington, D.C. Features include geofencing, grid-based clustering, and service radius checks to identify high-demand, underserved areas.

Help to determine the next place to build a bike station
To help Capital Bikeshare find the best spots for new bike stations, I used trip data and geospatial analysis. The goal was simple: pinpoint high-demand areas that current stations don’t cover well. The results show exactly where new stations would get the most use, improving service for riders.

Datasets
I used trip data from Capital Bikeshare’s public system, focusing on the last 3 months. The dataset, available [here](https://capitalbikeshare.com/system-data), includes trip start and end times, durations, and station locations, offering a complete view of recent bike usage in Washington, D.C.

Data Loading and Cleaning
I used Google Colab to run the analysis, leveraging its cloud-based environment for efficient data handling. The data was loaded into Pandas DataFrames from CSV files for each month. I merged the datasets into a single DataFrame and cleaned the data by removing rows with missing station names. Column names were also standardized for consistency.

