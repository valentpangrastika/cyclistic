
# Case Study - Cyclistic
## Initial Data Cleaning in Excel
We will be using the data from divvytrip that is available on https://divvy-tripdata.s3.amazonaws.com/index.html.
The data has been made available by Motivate International Inc. 
We will be using data from July 2022 to June 2023.

The data is organized in a structured format with rows and columns. 
Each row corresponds to a trip, and every trip is uniquely identified by a field called "ride_id."
Each file contain the following column :
* ride_id               #Ride id - unique
* rideable_type         #Bike type - Classic, Docked, Electric
* started_at            #Trip start day and time
* ended_at              #Trip end day and time
* start_station_name    #Trip start station
* start_station_id      #Trip start station id
* end_station_name      #Trip end station
* end_station_id        #Trip end station id
* start_lat             #Trip start latitude  
* start_lng             #Trip start longitute   
* end_lat               #Trip end latitude  
* end_lat               #Trip end longitute   
* member_casual         #Rider type - Member or Casual  

Our following action involves ensuring that the data is stored correctly and ready for analysis. 
To achieve this, I completed several tasks. 
Initially, I downloaded all 12 zip files and extracted their contents, placing them in a temporary folder on my desktop. 
Additionally, I established subfolders for both the .CSV and .XLS files to maintain original data copies.

For each .XLS file, I followed the subsequent steps:
1. Revised the format of the "started_at" and "ended_at" columns to a custom DATETIME formatted it using the pattern "yyyy-mm-dd h:mm:ss."
2. Introduced a new column named "ride_length" to calculate the duration of each ride by subtracting the "started_at" column from the "ended_at" column.
   represented it as TIME in "HH:MM:SS" format.
3. Created a "ride_date" column to compute the starting date of each ride using the DATE command, with the resulting dates formatted as "YYYY-MM-DD."
4. Established a "ride_month" column to record the month of each ride numerically (e.g., January as "1"), and set the format to Number.
5. Introduced a "ride_year" column to capture the year of each ride and maintained a general format ("YYYY").
6. Generated a "start_time" column to calculate the ride's start time based on the "started_at" column, formatted as TIME in "HH:MM:SS".
7. Generated an "end_time" column to determine the ride's end time using the "ended_at" column, also presented in TIME format ("HH:MM:SS").
8. Included a "day_of_week" column to compute the day of the week when each ride commenced using the WEEKDAY command, with the result presented as a whole number (e.g., 1 for Sunday and 7 for Saturday).
Following these modifications, I proceeded to save each .XLS file to avoid missing data.

## Data Cleaning in BigQuery
### Checking Data Loss and Duplicate
Initially, I transformed our .XLS file into a .CSV format and subsequently imported it into BigQuery. 
To ensure data integrity and detect any potential loss during this transformation, I executed both count and count distinct commands to examine duplicate entries and calculate the total number of rows.

```sql
SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202207_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202208_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202209_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202210_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202211_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202212_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202301_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202301_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202302_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202303_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202304_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202305_clean`;

SELECT COUNT(ride_id),
COUNT(DISTINCT(ride_id))
FROM `firm-aria-392507.Bike_Project.202306_clean`;

```
### Combining All Files
In order to perform the analysis by season, I combined these tables and saved them as "combined".
``` sql
SELECT *
FROM `firm-aria-392507.Bike_Project.202207_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202208_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202209_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202210_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202211_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202212_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202301_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202302_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202303_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202304_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202305_clean`

UNION DISTINCT

SELECT *
FROM `firm-aria-392507.Bike_Project.202306_clean`
```
### Perform Cleaning and Modification For day_of_week Column
Initially, I converted the data type of "day_of_week" from FLOAT to STRING. 
Subsequently, I replaced the numeric values with their respective day names (e.g., 1 will be transformed into "Sunday," and 7 will be changed to "Saturday").
Additionally, I introduced a new "month_year" column to signify the specific month and year.
``` sql
SELECT
ride_id,
rideable_type, 
started_at, 
ended_at,
ride_full_date,
month_year,
ride_year,
FORMAT_DATE('%m-%Y', STARTED_AT) AS month_year,
CASE 
  WHEN day_of_week= 1 THEN 'Sunday'
  WHEN day_of_week= 2 THEN 'Monday'
  WHEN day_of_week= 3 THEN 'Tuesday'
  WHEN day_of_week= 4 THEN 'Wednesday'
  WHEN day_of_week= 5 THEN 'Thursday'
  WHEN day_of_week= 6 THEN 'Friday'
  WHEN day_of_week= 7 THEN 'Saturday'
  END AS day_of_week, 
start_time,
end_time,
CAST(ride_length AS STRING) AS ride_length,
start_station_name, 
start_station_id, 
end_station_name, 
end_station_id, 
start_lat, 
start_lng, 
end_lat, 
end_lng, 
member_casual
FROM 
`firm-aria-392507.Bike_Project.combined`
ORDER BY started_at
```
###
