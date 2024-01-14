# Cyclistic
Google data analytics capstone project

## Introduction
This is my case study project for the [Google Data Analytics Professional Certificate published by coursera](https://www.coursera.org/learn/google-data-analytics-capstone). In this case study, I am performing a real-world task as a junior data analyst for the fictional company Cyclistic. For this project, I will work with a large dataset using Excel and MySQL to demonstrate my proficiency in data analysis. Additionally, I will utilize Power BI and PowerPoint to visualize and share the results of my analysis.

## Scenario
I am a junior data analyst in the marketing team at Cyclistic, a bike-share company in Chicago. The director of marketing believes that the company's future success depends on maximizing annual memberships. The goal is to understand the differences in how casual riders and annual members use Cyclistic bikes. From these insights, the team aims to design a new marketing strategy to convert casual riders into annual members.

Cyclistic launched a successful bike-share offering in 2016, growing to a fleet of 5,824 bicycles across 692 stations in Chicago. The marketing strategy has focused on building general awareness and appealing to broad consumer segments. The pricing plans include single-ride passes, full-day passes, and annual memberships. Annual members are identified as more profitable, prompting the need for strategies to convert casual riders. The director of marketing, Lily Moreno, sets a clear goal to design marketing strategies for this conversion Moreno and her team are interested in analyzing the Cyclistic historical bike trip data to identify trends. Shw, who is responsible for the development of campaigns and initiatives to promote the bike-share program, has assigned me a question to answer: How do annual members and casual riders use Cyclistic bikes differently?

## Analysis Process
To answer key business questions, I will follow the data analysis process: ask, prepare, process, analyze, share, and act. The deliverables include a clear statement of the business task, description of data sources, documentation of data cleaning or manipulation, summary of analysis, supporting visualizations, key findings, and recommendations.

## Analysis Steps

### 1- Ask
First, let’s identify the business task:
The business task is to figure out how casual riders and Cyclistic members use their rental bikes differently. From these insights, the team will design a new marketing strategy to convert casual riders into annual members.
Second, let’s designate key stakeholders:
In this case study our stakeholders are:
**Lily Moreno**: The director of marketing and my manager. Moreno is responsible for the development of campaigns and initiatives to promote the bike-share program. These may include email, social media, and other channels. 
**Cyclistic marketing analytics team**: A team of data analysts who are responsible for collecting, analyzing, and reporting data that helps guide Cyclistic marketing strategy. 
**Cyclistic executive team**: The notoriously detail-oriented executive team will decide whether to approve there commended marketing program.

### 2- Prepare
To answer the question, I will use Cyclistic’s historical trip data to analyze and identify trends and choose to work with first quarter of 2023 company’s data. 
### 2-1- Data Location and Credibility
I downloaded the data from [Divvy Bikes](https://divvy-tripdata.s3.amazonaws.com/index.html). with a license from [Motivate International Inc](https://divvybikes.com/data-license-agreement). This is public data that can be used to explore how different customer types are using Cyclistic bikes. Note that data-privacy issues prohibit users from using riders’ personally identifiable information. 
The company has their license for this data and removed any personal information for sake of privacy. Therefore, data is unbiased and ROCCC (means it is reliable, original, comprehensive, current and cited.)

#### 2-2- Structure of data
My data contain 3 distinct .csv files, each of which corresponds to a month of 2023. Each CSV file contains hundreds of thousands of data within 13 columns: 
| ride_id |	rideable_type	| started_at | ended_at	| start_station_name	| start_station_id	| end_station_name	| end_station_id	| start_lat	| start_lng	| end_lat	| end_lng	| member_casual |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |

### 3- Process
#### 3-1- Laoding and combining data
To combine and clean the data, I used Oracle’s MySQL. After downloading my data, I placed them in my sec-file-location. By using the **SHOW VARIABLES LIKE**`secure_file_priv;` code, I found this location.
Then I started uploading my datasets into MySQL.
```
-- Loading CSV files into MySQL
LOAD DATA INFILE 'sec-file-location/202301_divvy_tripdata.csv'
INTO TABLE 202301_divvy_tripdata
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';

LOAD DATA INFILE 'sec-file-location/202302_divvy_tripdata.csv'
INTO TABLE 202302_divvy_tripdata
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';

LOAD DATA INFILE 'sec-file-location/202303_divvy_tripdata.csv'
INTO TABLE 202303_divvy_tripdata
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n';
```
After creating my tables, I combined these tables using the Union function and stored the result in a new table named **2023_tripdata_combined**.
```
-- Combine the three tables using UNION and store in a new table
DROP TABLE IF EXISTS 2023_tripdata_combined;
CREATE TABLE 2023_tripdata_combined AS
SELECT * FROM 202301_divvy_tripdata
UNION
SELECT * FROM 202302_divvy_tripdata
UNION
SELECT * FROM 202303_divvy_tripdata;
```
Next step is exploring and cleaning my data. But before cleaning and making any changes, I decided to create a backup of my combined table.
```
-- Create a copy of the combined table before cleaning data
CREATE TABLE 2023_tripdata_combined_copy AS
SELECT * FROM 2023_tripdata_combined;
```

#### 3-2- Cleaning data
Before cleaning my data, let's explore the data.
```
-- checking my columns
DESCRIBE 2023_tripdata_combined;

-- Check the total number of columns in the combined table
SELECT COUNT(*) AS total_columns
FROM information_schema.columns
WHERE table_name = '2023_tripdata_combined';

-- Check the total number of rows in the combined table
SELECT COUNT(*) AS total_rows
FROM 2023_tripdata_combined;
```
Results show there are 13 columns and 498,537 rows. Now I start to clean my data. First, checking for duplicates.
Because of the method used to combine three tables, there should not be any duplicate rows.
```
-- Check for duplicate rows in the combined table
SELECT COUNT(*) - COUNT(DISTINCT ride_id) AS duplicate_rows
FROM 2023_tripdata_combined;
-- There is no duplicates beacause of method i used to combine three tables
```
Next, check for null values in my table.
```
-- Count the number of null values in the combined table
SELECT
  SUM(CASE WHEN ride_id IS NULL THEN 1 ELSE 0 END) AS null_ride_id,
  SUM(CASE WHEN rideable_type IS NULL THEN 1 ELSE 0 END) AS null_rideable_type,
  SUM(CASE WHEN started_at IS NULL THEN 1 ELSE 0 END) AS null_started_at,
  SUM(CASE WHEN ended_at IS NULL THEN 1 ELSE 0 END) AS null_ended_at,
  SUM(CASE WHEN start_station_name IS NULL THEN 1 ELSE 0 END) AS null_start_station_name,
  SUM(CASE WHEN start_station_id IS NULL THEN 1 ELSE 0 END) AS null_start_station_id,
  SUM(CASE WHEN end_station_name IS NULL THEN 1 ELSE 0 END) AS null_end_station_name,
  SUM(CASE WHEN end_station_id IS NULL THEN 1 ELSE 0 END) AS null_end_station_id,
  SUM(CASE WHEN start_lat IS NULL THEN 1 ELSE 0 END) AS null_start_lat,
  SUM(CASE WHEN start_lng IS NULL THEN 1 ELSE 0 END) AS null_start_lng,
  SUM(CASE WHEN end_lat IS NULL THEN 1 ELSE 0 END) AS null_end_lat,
  SUM(CASE WHEN end_lng IS NULL THEN 1 ELSE 0 END) AS null_end_lng,
  SUM(CASE WHEN member_casual IS NULL THEN 1 ELSE 0 END) AS null_member_casual
FROM 2023_tripdata_combined;
```
The result shows that there are no null values in my dataset. That is strange because, by looking at CSV files, I see some blanks. So, I decided to search for blank cells in my combined table.
```
-- Count rows with blank spaces in any column
SELECT COUNT(*)
FROM 2023_tripdata_combined
WHERE
  COALESCE(ride_id, '') = '' OR
  COALESCE(rideable_type, '') = '' OR
  COALESCE(started_at, '') = '' OR
  COALESCE(ended_at, '') = '' OR
  COALESCE(start_station_name, '') = '' OR
  COALESCE(start_station_id, '') = '' OR
  COALESCE(end_station_name, '') = '' OR
  COALESCE(end_station_id, '') = '' OR
  COALESCE(start_lat, '') = '' OR
  COALESCE(start_lng, '') = '' OR
  COALESCE(end_lat, '') = '' OR
  COALESCE(end_lng, '') = '' OR
  COALESCE(member_casual, '') = '';

-- Rows with blank spaces in any column
SELECT *
FROM 2023_tripdata_combined
WHERE
  COALESCE(ride_id, '') = '' OR
  COALESCE(rideable_type, '') = '' OR
  COALESCE(started_at, '') = '' OR
  COALESCE(ended_at, '') = '' OR
  COALESCE(start_station_name, '') = '' OR
  COALESCE(start_station_id, '') = '' OR
  COALESCE(end_station_name, '') = '' OR
  COALESCE(end_station_id, '') = '' OR
  COALESCE(start_lat, '') = '' OR
  COALESCE(start_lng, '') = '' OR
  COALESCE(end_lat, '') = '' OR
  COALESCE(end_lng, '') = '' OR
  COALESCE(member_casual, '') = '';
```
Results show there are 141,133 rows with blank spaces. All blanks are for *start_station_name* and *start_station_id* columns. Because there are lots of blank spaces, and also station name and ID are not too important for our analysis purpose, I decided to leave them blank. 
By looking at my dataset, I realized that *start_at* and *end_at* columns contain both data and time in the same cell. For finding how each user type is using the bike, it's appropriate to separate them into two columns.
```
-- Create a new table with separate date and time columns
CREATE TABLE 2023_tripdata_cleaned AS
SELECT
  ride_id,
  rideable_type,
  DATE(started_at) AS start_date,
  TIME(started_at) AS start_time,
  DATE(ended_at) AS end_date,
  TIME(ended_at) AS end_time,
  start_station_name,
  start_station_id,
  end_station_name,
  end_station_id,
  start_lat,
  start_lng,
  end_lat,
  end_lng,
  member_casual
FROM 2023_tripdata_combined;
```
Now it's easier to calculate duration of trips in minutes using *start_time* and *end_time*.
```
--             ---------------------------              --
-- Creating a columns to calculate the minutes of trips --
--             ---------------------------              --
-- Add a new column to calculate the duration in minutes
ALTER TABLE 2023_tripdata_cleaned
ADD COLUMN duration_min INT;

-- Update the new column with the duration in minutes
UPDATE 2023_tripdata_cleaned
SET duration_min = TIMESTAMPDIFF(MINUTE, start_time, end_time);
```
AAnd for the last cleaning process, I decided to change the column name from  *casual_member* to *user_type* for clarity.
```
-- Rename the column from member_casual to user_type
ALTER TABLE 2023_tripdata_cleaned
CHANGE COLUMN member_casual user_type VARCHAR(255);
```

### 4- Analysis
After cleaning my data I connected Powert BI to MySQL server to uplaod my cleaned table and start making charts and graph and do analysis.
![card](image.jpg)
Fist take a look at ***total trips*** and ***avarege of trips duration in minutes***.
Now let's make pie and donut charts summaring rides per hour.
![donut](image.jpg)
As the charts show **members** do did more trips than **casual** users. Also just a little amount of trips happed by **docked bikes**. But *average of minutes traveled* chart shows some intresting results. **casual** users put more minutes on ride a bike in average than **members** and also minutes traveled with **docked bikes** was significant longer than other rideable types. These need more investigsting. So let's take a look at *total tripes for each user type.
![each month](image.jpg)
Aha! Longer rides for **casual** members and **docked bikes** is due to thier limited useage of bike compared to **members** and other rideable types.
The bar graph above shows **members** never used **docked bikes** and they used **classic bikes** more than **elecrtic bikes**. However, **casual** users utilized **classic bikes** and **electric bikes** fairly similar.
Also, the chart shows both users were more active in March. It might be due to weather condition which March s a little bit warmer than other two.
![each day](image.jpg)
For this columns cart i made add a calculated column with Power BI to calculate each day name by using this code `DayName = FORMAT('cyclistic 2023_tripdata_combined'[end_at_date], "dddd")` . The results indicates that **casual** users traveled each day of the week fairly similar. Although, **members** ride more in weekdays. We can coclude that **members** use bike to get to their work more than just ride them as an excersize or for fun.
![stations](image.jpg)
![stations](image.jpg)
This maps shows that most papolar stations were in North East and South East.

### 5- Share
To share my insights I made [this](file) intractive powerpoint presentation.

### 6- Act
After the data has been processed, analyzed and insights have been shared, it's time to recommendintions for cyclistinc. My recommendations are:
1. Cyclistinc and make ads and campaigns near popular stations that could attract more casual riders and encourage them to become members. This ads can illustrate users could ride bike to get to their work and get rid of raffic hams and have healthy body.
2. Cyclistic can introduce a new monthly plan and  discounts for riders. The results of analysis indicated the number of rides affected the weather condition. Marketing campaignsin the warmer months could lead to a higher conversion rate.

## Conclusion
In this project, I assumed to work with cyclistic, a bike share company in Chiacgo. I used MySQL to prepare and clean my data analyze and used Power BI to visualize data, as well as draw meaningful insights from it to answer business task: how casual riders and Cyclistic members use their rental bikes differently. At the end, I gave my reommendations to the team to design a new marketing strategy to convert casual riders into annual members.
Thanks for your time to read my Capstone Project!
