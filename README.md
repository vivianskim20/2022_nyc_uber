# 2022 NYC Uber Analysis Overview

The preview version with only January 2022 data is available on [Tableau Public](https://public.tableau.com/app/profile/vivian.k4558/viz/shared/7J4PNRK8X).

## I. Objective

The purpose is to create an interactive dashboard that can allow users to run analysis on revenue/profit of Uber trips made in NYC in 2022. Depending on the time of the day, month of the year, pickup and dropoff location, and other various factors, the revenue differed greatly. With time and location breakdowns, users can have a better understanding of the fluctuation of the revenue/profit.

## II. High Level Requirement

| Requirement | Description |
| --- | ----------- |
| Allow the user to see the overview/summary of all the records | Total number of trips, total distance traveled, total time traveled, and total revenue/profit |
| Show revenue fluctuation by month, date, and time | Include selectors for line charts to give the user the option to pick month or date |
| Give a context about the KPI how it was computed | Text boxes/annotations with detailed explanations on the computation under each section |

## III. Metrics

The following metrics are used to describe the sales per breakdown. 
- **Total number of trips**: total count of trips completed in 2022
- **Total/average revenue**: total revenue earned/average revenue earned per trip
  - Fare including taxes, surcharges, tolls, and/or tips 
- **Total/average profit**: total profit made/average profit made per trip
  - Driver compensations subtracted from fare excluding taxes, surcharges, tolls, and/or tips
    - Driver compensation includes tips
    - According to this article from Uber, it heavily depends on the situation whether Uber gets to keep surcharges and tolls, so tolls are not included in profit to best describe the actual profit
- **Gross profit margin**: percentage of profit made from revenue
- **Average driver earning**: average driver compensation (driver earning net of commission, surcharge, and/or tax plus tips) per trip
- **Total surcharges**: total amount collected in trip for Black Car Fund, total amount collected in trip for NYS congestion surcharge, and $2.50 for both drop off and pick up at LaGuardia, Newark, and John F. Kennedy airports
- **Surcharge percentage**: average percentage of surcharge in revenue

These metrics have breakdowns by month, day, and time to allow users to compare the differences and better understand the fluctuations. See below for initial prototype of the dashboard:

<p align="center">
<img src="https://github.com/vivianskim20/2022_nyc_uber/blob/main/dashboard%20plan.jpeg" alt="drawing" width="545" height="800"/>
</p>

## IV. Data Sources

The datasets used in this dashboard are from the following sources:
- [NYC Taxi and Limousine Commission](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
  - Monthly files of high-volume for-hire vehicle trip records of 2022
  - A lookup table for taxi zones 
  - A data dictionary document with detailed explanations of each column
- [NYC Taxi Zones](https://data.cityofnewyork.us/Transportation/NYC-Taxi-Zones/d3c5-ddgc)
  - Longitude and latitude of the taxi zones appeared in trip records
 
## V. Infrastructure

<p align="center">
<img src="https://github.com/vivianskim20/2022_nyc_uber/blob/main/process.png" alt="drawing" width="402" height="300"/>
</p>

From the data sources, data files are obtained and processed in Jupyter Notebook in the following order:
- Trip data
  1. The 12-month trip files in PARQUET are converted to data frames in pandas
  2. According to the data dictionary provided by NYC TLC, a filter is applied to the column ‘hvfhs_license_num’ to only keep those records of Uber
  3. The resulting rows are concatenated to make a single CSV file that contains all rows and columns of interest
- Taxi zone data
  1. The column ‘the_geom’ contains all combinations of latitude and longitude for the specific zone-borough, so the average of all latitudes and all longitudes are computed to give each zone-borough a unique latitude and longitude
Since ‘LocationID’ has duplicate rows, it’s dropped and ‘OBJECTID’ is renamed to ‘LocationID’ for readability
The columns ‘zone’ and ‘borough’ are concatenated with a comma in the middle to later show them in an interactive map
The final result is exported to a CSV file

In Snowflake, four fact and dimension tables are created according to the data schema shown below. The resulting tables are loaded to Tableau and joined to create a dashboard.

## VI. Objects

1. Data schema:

<p align="center">
<img src="https://github.com/vivianskim20/2022_nyc_uber/blob/main/data%20schema.png" alt="drawing" width="757" height="400"/>
</p>

2. fct_trips:

| Parameter | Data type | Description |
| ----------| --------- | ----------- |
|request_datetime|datetime|Date/time stamp of the request|
|PULocationID|int|TLC Taxi Zone in which the trip began|
|DOLocationID|int|TLC Taxi Zone in which the trip ended|
|trip_miles|double|Total distance traveled in miles in the trip|
|trip_time|int|Total time spent in the trip in seconds|
|base_passenger_fare|double|Base passenger fare before tolls, tips, taxes, and fees|
|tolls|double|Total amount of all tolls paid in the trip|
|sales_tax|double|Total amount collected in trip for NYS sales tax|
|surcharges|double|Total amount collected in trip for Black Car Fund + total amount collected in trip for NYS congestion surcharge + $2.50 for both drop off and pick up at LaGuardia, Newark, and John F. Kennedy airports|
|tips|double|Total amount of tips received from passenger|
|driver_pay|double|Total driver pay (not including tolls or tips and net of commission, surcharge, or tax)|

3. dim_request_datetime

| Parameter   | Data type | Description |
| ----------- | --------- | ----------- |
|**request_datetime**|datetime|Date/time stamp of the request|
|request_month|varchar|Month of the request (ie. Jan., Feb., …)|
|request_date|datetime|Date stamp of the request (ie. 2022-01-01)|
|request_hour|datetime|Hour of the request (ie. 2022-01-01 12:00:00)|

4. dim_pickup_loc
   
| Parameter   | Data type | Description |
| ----------- | --------- | ----------- |
|**PULocationID**|varchar|Identification of the pick up location|
|zone_borough|varchar|Zone and borough of the pick up location separated by a comma|
|latitude|double|Latitude of the pick up location|
|longitude|double|Longitude of the pick up location|

5. dim_dropoff_loc
   
| Parameter   | Data type | Description |
| ----------- | --------- | ----------- |
|**DOLocationID**|varchar|Identification of the drop off location|
|zone_borough|varchar|Zone and borough of the drop off location separated by a comma|
|latitude|double|Latitude of the drop off location|
|longitude|double|Longitude of the drop off location|


