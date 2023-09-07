# Data Exploration and Analysis
## The analysis will steer us toward answering the following question: How do annual members and casual riders use Cyclistic bikes differently?
## Explore The Distribution of Members and Casual Riders 
The query's objective is to provide insights into the membership composition of bike riders on a monthly basis.
We can infer that the query results show that the largest number of riders occur in the month of July 2022, while the lowest number of riders occur in the month of December.
Additionally, highest member and casual percentage happened on respectively January 2023 and July 2022.
``` sql
WITH 
    temp_1 AS 
    (
        SELECT
            month_year,
            COUNT(member_casual) as total_count
        FROM `firm-aria-392507.Bike_Project.bike_clean`
        GROUP BY month_year
    ),
    temp_2 AS
    (
        SELECT 
            month_year,
            COUNT(member_casual) AS member_count
        FROM `firm-aria-392507.Bike_Project.bike_clean`
        WHERE member_casual = "member"
        GROUP BY month_year
    ),
    temp_3 AS
    (
        SELECT
            month_year,
            COUNT(member_casual) AS casual_count
        FROM `firm-aria-392507.Bike_Project.bike_clean`
        WHERE member_casual = "casual"
        GROUP BY month_year
    )

SELECT
    temp_1.month_year,
    temp_2.member_count,
    ROUND(temp_2.member_count / temp_1.total_count * 100, 2) AS member_percentage,
    temp_3.casual_count,
    ROUND(temp_3.casual_count / temp_1.total_count * 100, 2) AS casual_percentage,
    temp_1.total_count
FROM temp_1
INNER JOIN temp_2 ON temp_1.month_year = temp_2.month_year
INNER JOIN temp_3 ON temp_2.month_year = temp_3.month_year


```
![percentage](https://github.com/valentpangrastika/images/blob/48267f78480f3d0e4c2473a1154b525596102f55/Screen%20Shot%202023-09-06%20at%2016.47.42.png)

## Explore The Average Ride Length
In this query, a comparative analysis is conducted to examine the average ride lengths of two distinct groups of riders: "members" and "casual riders." The purpose is to evaluate and contrast the typical ride durations of these two rider categories across various months and years. Based on the findings, it consistently appeared that casual riders tend to have longer ride durations compared to members.
``` sql
WITH temp_1 AS (
    SELECT 
        month_year,
        member_casual,
        AVG(CAST(ride_length AS INTERVAL)) AS average_member_ride_length
    FROM `firm-aria-392507.Bike_Project.bike_clean`
    WHERE member_casual = "member"
    GROUP BY month_year, member_casual
),
temp_2 AS (
    SELECT 
        month_year,
        member_casual,
        AVG(CAST(ride_length AS INTERVAL)) AS average_casual_ride_length
    FROM `firm-aria-392507.Bike_Project.bike_clean`
    WHERE member_casual = "casual"
    GROUP BY month_year, member_casual
)
SELECT
    temp_1.month_year,
    temp_1.average_member_ride_length,
    temp_2.average_casual_ride_length
FROM temp_1
JOIN temp_2 ON temp_1.month_year = temp_2.month_year;
```
![avg_ride_length](https://github.com/valentpangrastika/images/blob/42bf32b1d810385cb78e348b67dbfcc9188dbaf7/Screen%20Shot%202023-09-06%20at%2017.19.26.png)

### Checking "ride_length" outliers
To assess the presence of outliers in the "ride_length" data, we examined the highest recorded value within the ride_length column.
This process helped us establish a criterion where data points exceeding this threshold were regarded as exceptional or extreme.
Based on the findings, the maximum ride_length for both member and casual riders in each month consistently fell within the range of 22 to 23 hours.
``` sql
WITH MaxRideLengths AS (
  SELECT
    month_year,
    member_casual,
    MAX(CAST(ride_length AS INTERVAL)) AS max_ride_length
  FROM
    `firm-aria-392507.Bike_Project.bike_clean`
  GROUP BY
    month_year, member_casual
)

SELECT
  month_year,
  MAX(CASE WHEN member_casual = 'member' THEN max_ride_length END) AS max_member_ride_length,
  MAX(CASE WHEN member_casual = 'casual' THEN max_ride_length END) AS max_casual_ride_length
FROM
  MaxRideLengths
GROUP BY
  month_year
ORDER BY
  month_year;

```
![](https://github.com/valentpangrastika/images/blob/50397ed8973c84b20e6f7c84e205acd8c57c29c2/Screen%20Shot%202023-09-06%20at%2018.55.01.png)
Here, we reevaluated the number of riders who fall within those extreme ride length ranges.
Based on the findings, there are relatively few instances of riders with exceptionally long ride lengths. Consequently, I have chosen to incorporate these cases into our future analysis and proceed with the assumption that there are no outliers.
``` sql
SELECT
FORMAT_TIMESTAMP('%Y-%m-%d %H:%M', PARSE_TIMESTAMP('%H:%M:%S', CAST(ride_length AS STRING))) AS truncated_min,
COUNT(*) AS total_trips,
SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips,
SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips
FROM
`firm-aria-392507.Bike_Project.bike_clean`
GROUP BY
truncated_min
ORDER BY
total_trips DESC;
```
![](https://github.com/valentpangrastika/images/blob/17dcae5cbd13ab36002681eb6a46fcce360df880/Screen%20Shot%202023-09-06%20at%2018.56.52.png)

![](https://github.com/valentpangrastika/images/blob/17dcae5cbd13ab36002681eb6a46fcce360df880/Screen%20Shot%202023-09-06%20at%2018.57.40.png)

## Checking The Busiest Day Based On the Num of Riders
Analyzing the busiest day based on the number of riders involves a thorough examination of data related to the volume of riders on each day. Based on the result, member riders tend to favor weekday. Meanwhile casual riders favor weekend. 
``` sql
SELECT
COUNT(ride_id) AS total_trips,
SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips,
SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips,
day_of_week,
FROM `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY day_of_week
ORDER BY total_trips DESC
```
![](https://github.com/valentpangrastika/images/blob/2ce3da4541cd2fa71abdbff267c0703e87dec74f/Screen%20Shot%202023-09-06%20at%2019.01.14.png)

## Checking The Top 10 Most Frequented Station
In this analysis, we identify the locations where bike demand is at its peak, both at the beginning of rides (start stations) and at the conclusion of journeys (end stations). This knowledge enables us to fine-tune the distribution of bikes among stations, guaranteeing that areas with the greatest demand receive a sufficient supply. Additionally, we gain deeper understanding of how users navigate the city. 
### Start Station
``` sql
SELECT DISTINCT(start_station_name),
COUNT(member_casual) AS total_trips,
SUM(CASE WHEN member_casual="member" THEN 1 ELSE 0 END) AS member_trips,
SUM(CASE WHEN member_casual="casual" THEN 1 ELSE 0 END) AS casual_trips
FROM `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY start_station_name
ORDER BY total_trips DESC
LIMIT 10
```
![](https://github.com/valentpangrastika/images/blob/e75ee51a23abcbe25d8c55b48190189f7ba39067/Screen%20Shot%202023-09-07%20at%2013.29.50.png)

### End Station
``` sql
SELECT DISTINCT(end_station_name),
COUNT(member_casual) AS total_trips,
SUM(CASE WHEN member_casual="member" THEN 1 ELSE 0 END) AS member_trips,
SUM(CASE WHEN member_casual="casual" THEN 1 ELSE 0 END) AS casual_trips
FROM `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY end_station_name
ORDER BY total_trips DESC
LIMIT 10
```

![](https://github.com/valentpangrastika/images/blob/e75ee51a23abcbe25d8c55b48190189f7ba39067/Screen%20Shot%202023-09-07%20at%2013.30.09.png)

### Path (Start Station- End Station)

``` sql
SELECT
  CONCAT(start_station_name, ' to ', end_station_name) AS path,
  COUNT(*) AS total_trips,
  SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips,
  SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips
FROM `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY path
ORDER BY total_trips DESC
LIMIT 10
```

![](https://github.com/valentpangrastika/images/blob/8b6d7fa9c6ea8d0de2d4853b735ca6581f2b9dbb/Screen%20Shot%202023-09-07%20at%2013.37.59.png)

## Checking The Busiest Time of Day
Here we indentify the peak usage times for us to gain valuable insights into user conduct and utilization patterns. This data helps us to recognize trends, pinpoint high-demand hours, and identify opportunities for optimizing services.

``` sql
SELECT
    TIME(DATE_TRUNC(started_at, hour)) AS truncated_hour,
    COUNT(TIME(DATE_TRUNC(started_at, hour))) AS total_trips,
    SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips,
    SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips
FROM
    `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY
    truncated_hour
ORDER BY
    total_trips DESC ;
````
![](https://github.com/valentpangrastika/images/blob/c6d0c67a465cd754ac2550d665a550b89cdc4ead/Screen%20Shot%202023-09-07%20at%2013.34.41.png)

### Checking The Type of Bike 
Here we perform an analysis on the dataset of bike trips, specifically to determine which types of bikes (classic, electric, docked) are more favored by users categorized as "member" and "casual".
``` sql
SELECT
member_casual,
SUM(CASE WHEN rideable_type = 'classic_bike' THEN 1 ELSE 0 END) AS classic_trips,
SUM(CASE WHEN rideable_type = 'electric_bike' THEN 1 ELSE 0 END) AS electric_trips,
SUM(CASE WHEN rideable_type = 'docked_bike' THEN 1 ELSE 0 END) AS docked_trips
FROM
`firm-aria-392507.Bike_Project.bike_clean`
GROUP BY member_casual
```

![](https://github.com/valentpangrastika/images/blob/08c927b4300c89460638e073772d34ebd6465493/Screen%20Shot%202023-09-07%20at%2013.40.22.png)

