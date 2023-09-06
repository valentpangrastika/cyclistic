# Data Exploration and Analysis
## Explore The Distribution of Members and Casual Riders 
The query's objective is to provide insights into the membership composition of bike riders on a monthly basis.
We can infer that the query results show that the largest number of riders occur in the month of July 2022, while the lowest number of riders occur in the month of December.
Additionally, highest member and casual percentage happened on respectively January 2023 and July 2022.
``` sql
WITH 
temp_1 AS 
(SELECT
month_year,
COUNT(member_casual) as total_count
FROM `firm-aria-392507.Bike_Project.bike_clean`
GROUP BY month_year),
temp_2 AS
(SELECT 
month_year,
COUNT(member_casual) AS member_count
FROM `firm-aria-392507.Bike_Project.bike_clean`
WHERE member_casual= "member"
GROUP BY month_year),
temp_3 AS
(SELECT
month_year,
COUNT(member_casual) AS casual_count
FROM `firm-aria-392507.Bike_Project.bike_clean`
WHERE member_casual= "casual"
GROUP BY month_year)

SELECT
temp_1.month_year,
temp_2.member_count,
ROUND(temp_2.member_count/temp_1.total_count*100,2) AS member_percentage,
temp_3.casual_count,
ROUND(temp_3.casual_count/temp_1.total_count*100,2)AS casual_percentage,
temp_1.total_count
FROM temp_1
INNER JOIN temp_2 ON temp_1.month_year=temp_2.month_year
INNER JOIN temp_3 ON temp_2.month_year=temp_3.month_year
```
## 

