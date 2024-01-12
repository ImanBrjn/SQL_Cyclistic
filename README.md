# Cylictic
Google data analytics capstone project

## Introduction
This is my case study project for the [Google Data Analytics Professional Certificate published by coursera](https://www.coursera.org/learn/google-data-analytics-capstone). In this case study, I am performing a real-world task of a junior data analyst and working for a fictional company, Cyclistic. In this project, I will be working with a large dataset using Excel ad MySQL to demonstrate my proficiency in data analysis and using Power BI and Powerpoint to visualize and share my analysis results.

## Scenario
In this case study, I am a junior data analyst working in the marketing analyst team at *Cyclistic*, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, analysis team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, the team will design a new marketing strategy to convert casual riders into annual members. 

## Characters and teams 
In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to any other station in the system anytime. Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One 2 approach that helped make these things possible was the flexibility of its pricing plans: single-ride passes, full-day passes, and annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who purchase annual memberships are Cyclistic members. Cyclistic’s finance analysts have concluded that annual members are much more profitable than casual riders. Although the pricing flexibility helps Cyclistic attract more customers, Moreno believes that maximizing the number of annual members will be key to future growth. Rather than creating a marketing campaign that targets all-new customers, Moreno, the director of marketing and my manager, believes there is a very good chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have chosen Cyclistic for their mobility needs. Moreno has set a clear goal: Design marketing strategies aimed at converting casual riders into annual members. In order to do that, however, the marketing analyst team needs to better understand how annual members and casual riders diŦer, why casual riders would buy a membership, and how digital media could affect their marketing tactics. Moreno and her team are interested in analyzing the Cyclistic historical bike trip data to identify trends
In order to answer the key business questions, I will follow the steps of the data analysis process: ask, prepare, process, analyze, share, and act a produce a report with the following deliverables:
1.	A clear statement of the business task
2.	A description of all data sources used
3.	Documentation of any cleaning or manipulation of data 
4.	A summary of analysis 
5.	Supporting visualizations and key findings 
6.	Recommendations based on my analysis
Moreno, who is responsible for the development of campaigns and initiatives to promote the bike-share program, has assigned me a question to answer: How do annual members and casual riders use Cyclistic bikes differently?
Let's start do analysis step by step.

## 1- Ask
First, let’s identify the business task:
The business task is to figure out how casual riders and Cyclistic members use their rental bikes differently. From these insights, the team will design a new marketing strategy to convert casual riders into annual members.
Second, let’s designate key stakeholders:
In this case study our stakeholders are:
**Lily Moreno**: The director of marketing and my manager. Moreno is responsible for the development of campaigns and initiatives to promote the bike-share program. These may include email, social media, and other channels. 
**Cyclistic marketing analytics team**: A team of data analysts who are responsible for collecting, analyzing, and reporting data that helps guide Cyclistic marketing strategy.  
**Cyclistic executive team**: The notoriously detail-oriented executive team will decide whether to approve there commended marketing program.

## 2- Prepare
To answer the question, I will use Cyclistic’s historical trip data to analyze and identify trends and choose to work with first quarter of 2023 company’s data. 
### 2-1- Data Location and Credibility
I downloaded the data from [here](https://divvy-tripdata.s3.amazonaws.com/index.html). The data has been made available by ***Motivate International Inc***. under [this license](https://divvybikes.com/data-license-agreement). This is public data that can be used to explore how different customer types are using Cyclistic bikes. Note that data-privacy issues prohibit users from using riders’ personally identifiable information. 
The company has their license for this data and removed any personal information for sake of privacy. Therefore, data is unbiased and ROCCC (means it is reliable, original, comprehensive, current and cited.)

### 2-2- Data Organization
My data contain 3 distinct .csv files, each of which corresponds to a month of 2023. Each CSV file contains hundreds of thousands of data within 13 columns: 
| ride_id |	rideable_type	| started_at | ended_at	| start_station_name	| start_station_id	| end_station_name	| end_station_id	| start_lat	| start_lng	| end_lat	| end_lng	| member_casual |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |

## 3- Process
To combine and clean the data I used Oracle’s MySQL. After downloading my data in put them in my **sec-file-location**. By using `SHOW VARIABLES LIKE 'secure_file_priv';
` code I found this location.
Then i start to upload my datasets in MySQL.
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
Afther creating my tables, I combine these tables using Uninon function and store them in a new table name **2023_tripdata_combined**.
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
Next step is exploring and cleaning my data. But before cleaning and make any changes Idescided to create a back up from my combined table.
```
-- Create a copy of the combined table before cleaning data
CREATE TABLE 2023_tripdata_combined_copy AS
SELECT * FROM 2023_tripdata_combined;
```

## 4 - Cleaning data
Before cleaning my data I descided to explore my data.
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
Results show there are 13 columns and 498537 rows.
Now I start to clean my data.
First ckechikng for duplicates. Because of methoed used to combine three tables, should not be any duplicate rows. 
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
Result shows there is no null values in my dataset! That is strange! Because by looking at csv files I see some blanks.
So i descided to search for blank cells in my combined table.
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
Results show there are 141133 rows with blank spaces. All blanks are for start_station_name and start staion_id columns
Because there are lots of blank spaces and also station name and id are not too important for our analysis purpose, I descided to leave them blank.



