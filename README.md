# Bellabeat Case Study
Author: Olivia Chan  
Date: 1/15/2024
## Table of Contents
This case study follows the 6-step data analysis process:    
    &nbsp;&nbsp;&nbsp;&nbsp;I) Ask  
    &nbsp;&nbsp;&nbsp;&nbsp;II) Prepare  
    &nbsp;&nbsp;&nbsp;&nbsp;III) Process  
    &nbsp;&nbsp;&nbsp;&nbsp;IV) Analyze  
    &nbsp;&nbsp;&nbsp;&nbsp;V) Share  
    &nbsp;&nbsp;&nbsp;&nbsp;VI) Act  
## Introduction 
&nbsp;&nbsp;&nbsp;&nbsp;Bellabeat is a women’s wellness company that designs wearable technology and digital products that empower women to take control of their health. Their most well known product is known as ‘Ivy’, a wrist tracker that is capable of recording biometrics such as heart rate, sleep patterns, and menstrual cycles. In this case study, the CCO of Bellabeat, Urška Sršen, has prompted an analysis of smart device fitness data in hopes of revealing key insights that may help influence Bellabeat’s marketing strategy.
## I) Ask
<em> Ask questions and define the problem. </em>  


Business Task:  
Analyze data surrounding smart device usage and analyze consumer behavior with non-Bellabeat smart devices.  


Focus Questions:  
&nbsp;&nbsp;&nbsp;&nbsp;- What are some trends in smart device usage?  
&nbsp;&nbsp;&nbsp;&nbsp;- How could these trends apply to Bellabeat customers?  
&nbsp;&nbsp;&nbsp;&nbsp;- How could these trends help influence Bellabeat marketing strategy?  

## II) Prepare
<em> Prepare data by collecting and storing the information.</em>  

For this case study, the data set being analyzed is the [FitBit Fitness Tracker Data - Mobius](https://www.kaggle.com/datasets/arashnic/fitbit)  
This Kaggle data set consists of 18 CSV files containing the personal fitness tracker data of 30 FitBit users. Let’s use the ROCC approach to test the data set’s credibility.  

<b> Reliable: </b> The data set is quite small with only 30 participants. It is not a good representation of the millions of existing FitBit users as a whole. The timeframe of data collection is also relatively short at just one month. Thus, it may be difficult to identify long-term trends and insights.  

<b> Original: </b> The data set is not original. The data is taken from a third-party Amazon Mechanical Turk survey.  

<b> Comprehensive: </b> Besides the data set being on the smaller/shorter side, the data set is also missing key information surrounding participant demographics (age, gender, geographical location, height, etc.). Additionally, there is no information surrounding how participants were chosen for this survey, which could introduce bias.  

<b> Current: </b> This data set is from March 2016 to May 2016 so it is not current. The data is 7 to 8 years old at this point.  

<b> Cited: </b> The data is cited.  

*Considering these limitations, it is not recommended that the use of this data set alone be relied upon for business considerations. It would be better to supplement this analysis with further study.*  

For this analysis, the following files will be analyzed:  
&nbsp;&nbsp;&nbsp;&nbsp;- dailyActivity_Merged    
&nbsp;&nbsp;&nbsp;&nbsp;- sleepDay_Merged  
&nbsp;&nbsp;&nbsp;&nbsp;- hourlyCalories_Merged  
&nbsp;&nbsp;&nbsp;&nbsp;- hourlySteps_Merged  
&nbsp;&nbsp;&nbsp;&nbsp;- hourlyIntensities_Merged  

## III) Process  
<em> Process data by cleaning and checking the information. </em>

For the first round of data cleansing, Excel will be used.  
&nbsp;&nbsp;&nbsp;&nbsp;- For the file sleepDay_Merged, change SleepDay to short date format in order to get rid of null value time “0:00:00”  
&nbsp;&nbsp;&nbsp;&nbsp;- Change the format of the ActivityHour column in the files hourlyCalories_merged, hourlySteps_merged, and hourlyIntensities_merged so that it’s easier to auto detect schema in Google Big Query later.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Separate Activity Hour into two columns, one with short format date and one with time.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- <img src="https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/09d1892f-2768-48f6-ad18-94f0e66cec36" height="250" width= "325">  

For the second round of data cleansing, SQL will be used.  

<ins>Checking the number of distinct IDs in each data set</ins>  

```sql
-- dailyActivity_Merged
SELECT COUNT(DISTINCT ID)
FROM `bellabeat-case-study-410302.activity.activity_daily`
```
```sql
-- hourlyCalories_Merged
SELECT COUNT(DISTINCT ID)
FROM `bellabeat-case-study-410302.calories.calories_hourly`
```
```sql
-- hourlyIntensities_Merged
SELECT COUNT(DISTINCT ID)
FROM `bellabeat-case-study-410302.intensities.intensities_hourly`
```
```sql
-- hourlySteps_Merged
SELECT COUNT(DISTINCT ID)
FROM `bellabeat-case-study-410302.steps.steps_hourly`
```
```sql
-- sleepDay_Merged
SELECT COUNT(DISTINCT ID)
FROM `bellabeat-case-study-410302.sleep.sleep_daily`
```

Results: Each data set has 33 unique IDs except for sleepDay_Merged, which has 24 IDs.  

<ins> Checking for duplicate entries across all data sets </ins>  
```sql
-- dailyActivity_Merged
SELECT Id, ActivityDate, COUNT(*) AS Occurrences
FROM `bellabeat-case-study-410302.activity.activity_daily`
GROUP BY Id, ActivityDate
HAVING COUNT(*) > 1;
```
```sql
-- hourlyCalories_Merged
SELECT Id, ActivityHour_Date, ActivityHour_Time, COUNT(*) AS Occurrences
FROM `bellabeat-case-study-410302.calories.calories_hourly`
GROUP BY Id, ActivityHour_Date, ActivityHour_Time
HAVING COUNT(*) > 1;
```
```sql
-- hourlyIntensities_Merged
SELECT Id, ActivityHour_Date, ActivityHour_Time, COUNT(*) AS Occurrences
FROM `bellabeat-case-study-410302.intensities.intensities_hourly`
GROUP BY Id, ActivityHour_Date, ActivityHour_Time
HAVING COUNT(*) > 1;
```
```sql
-- hourlySteps_Merged
SELECT Id, ActivityHour_Date, ActivityHour_Time, COUNT(*) AS Occurrences
FROM `bellabeat-case-study-410302.steps.steps_hourly`
GROUP BY Id, ActivityHour_Date, ActivityHour_Time
HAVING COUNT(*) > 1;
```
```sql
-- sleepDay_Merged
SELECT Id, SleepDay, COUNT(*) AS Occurrences
 FROM `bellabeat-case-study-410302.sleep.sleep_daily`
 GROUP BY Id, SleepDay
 HAVING COUNT(*) >1;
```
Results: No duplicates found except in sleepDay_Merged data (see below)  
![Screenshot 2024-01-15 170331](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/93995544-cf2e-478d-8111-f6dc839f51c5)  


<ins> Merge hourly data sets into one table for more efficient queries </ins>  
```sql
-- create new table merged.Hourly 
CREATE TABLE merged.Hourly (
  Id numeric (10,0),
  dates string (50),
  times int,
  calories numeric (5,0),
  total_intensity numeric (5,0),
  avg_intensity float64,
  total_steps numeric (5,0)
);

-- join individual hourly data tables together and map columns to the right columns in merged.Hourly
INSERT INTO merged.Hourly(
  Id, dates, times, calories, total_intensity, avg_intensity, total_steps)
  (SELECT cal.Id, cal.dates, cal.times, cal.calories, int.total_intensity, int.avg_intensity, step.total_steps
  FROM calories_hourly AS cal
  INNER JOIN intensities_hourly AS int
  ON cal.Id = int.Id AND cal.dates = int.dates AND cal.times = int.times
  INNER JOIN steps_hourly AS step
  ON cal.Id = step.Id AND cal.dates = step.dates AND cal.times = step.times);
```
*The hourly data sets can also be merged via Excel. First, create a primary key by using the CONCAT function between the Id and ActivityHour columns. Next, use the primary key in a VLOOKUP function to pull unique columns from each file into one merged table.  
![Screenshot 2024-01-07 024921](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/2d1e140f-a606-4581-be87-190b99bcc7c3)  

<ins> Merge daily data sets into one table for more efficient queries </ins> 
```sql
SELECT *
FROM `bellabeat-case-study-410302.activity.activity_daily` AS activity_daily
LEFT JOIN `bellabeat-case-study-410302.sleep.sleep_daily` AS sleep_daily
ON activity_daily.ActivityDate = sleep_daily.SleepDay AND activity_daily.Id = sleep_daily.Id
```

*This can also be done via Excel using a concatenated primary key and VLOOKUP.  
![Screenshot 2024-01-13 215120](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/7e39084f-ccd9-4baa-b1dd-0110be833f09)  
*Notice there are #N/A results from the VLOOKUP because the sleep data set only has data from 24 ID’s and not 33.  

## IV) Analyze  
<em> Analyze data to find patterns, relationships, and trends. </em>  

<ins> Average time spent for each activity level </ins>  
```sql
-- average time spent in minutes for each activity level
SELECT ROUND(AVG(activity_daily.VeryActiveMinutes),2) AS Avg_Very_Active_Minutes,
ROUND(AVG(activity_daily.FairlyActiveMinutes),2) AS Avg_Fairly_Active_Minutes,
ROUND(AVG(activity_daily.LightlyActiveMinutes),2)AS Avg_Lightly_Active_Minutes,
ROUND(AVG(activity_daily.SedentaryMinutes),2) AS Avg_Sedentary_Minutes
FROM `bellabeat-case-study-410302.activity.activity_daily` AS activity_daily
LEFT JOIN `bellabeat-case-study-410302.sleep.sleep_daily` AS sleep_daily
ON activity_daily.ActivityDate = sleep_daily.SleepDay AND activity_daily.Id = sleep_daily.Id
```
We can do this in hours too:
```sql
-- average time spent in hours for each activity level
SELECT ROUND(AVG(activity_daily.VeryActiveMinutes)/60, 2) AS Avg_Very_Active_Hours,
ROUND(AVG(activity_daily.FairlyActiveMinutes)/60,2) AS Avg_Fairly_Active_Hours,
ROUND(AVG(activity_daily.LightlyActiveMinutes)/60,2) AS Avg_Lightly_Active_Hours,
ROUND(AVG(activity_daily.SedentaryMinutes)/60,2) AS Avg_Sedentary_Hours
FROM `bellabeat-case-study-410302.activity.activity_daily` AS activity_daily
LEFT JOIN `bellabeat-case-study-410302.sleep.sleep_daily` AS sleep_daily
ON activity_daily.ActivityDate = sleep_daily.SleepDay AND activity_daily.Id = sleep_daily.Id
```
![Screenshot 2024-01-18 132936](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/d097acac-aecb-471b-ba5b-50dd4ce1285e)  

*Participants spent the most time engaging in sedentary activity with an average of 991.21 minutes or 16.51 hours a day. Coming in second, they also spent an average of 192.81 minutes or 3.22 hours a day engaging in light activity. The least amount of time was spent engaging in very active or fairly active activities with just 21.16 and 13.56 minutes being spent on average in a day, respectively.  

<ins> Average calories burned and average total steps taken </ins>  
```sql
-- average calories burned and average total steps taken
SELECT ROUND(AVG(activity_daily.Calories), 2) AS Avg_Calories_Burned,
ROUND(AVG(activity_daily.TotalSteps),2) AS Avg_Total_Steps
FROM `bellabeat-case-study-410302.activity.activity_daily` AS activity_daily
LEFT JOIN `bellabeat-case-study-410302.sleep.sleep_daily` AS sleep_daily
ON activity_daily.ActivityDate = sleep_daily.SleepDay AND activity_daily.Id = sleep_daily.Id
```
![Screenshot 2024-01-18 133205](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/98ee35c8-31eb-452e-bf08-b6deb948cfd5)  

*Participants burned an average of 2307.51 calories a day and took an average of 7652.19 steps a day.  

<ins> Average active distance as % of total distance </ins>  
```sql
-- Add columns to the table
ALTER TABLE activity.activity_daily
ADD COLUMN VAD_Ratio DECIMAL (10,2),
ADD COLUMN MAD_Ratio DECIMAL (10,2),
ADD COLUMN LAD_Ratio DECIMAL (10,2),
ADD COLUMN SAD_Ratio DECIMAL (10,2);


-- Update the values in the new columns
UPDATE activity.activity_daily
SET
  VAD_Ratio = activity_daily.VeryActiveDistance / activity_daily.TotalDistance,
  MAD_Ratio = activity_daily.ModeratelyActiveDistance / activity_daily.TotalDistance,
  LAD_Ratio = activity_daily.LightActiveDistance / activity_daily.TotalDistance,
  SAD_Ratio = activity_daily.SedentaryActiveDistance / activity_daily.TotalDistance;


-- Find the average active distance ratio
SELECT ROUND(AVG(VAD_Ratio), 2) AS Avg_VAD_Ratio,
ROUND(AVG(MAD_Ratio), 2) AS Avg_MAD_Ratio, 
ROUND(AVG(LAD_Ratio), 2) AS Avg_LAD_Ratio, 
ROUND(AVG(SAD_Ratio), 2) AS Avg_SAD_Ratio
FROM `bellabeat-case-study-410302.activity.activity_daily`
```
![Screenshot 2024-01-18 133303](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/637ad525-912e-4abc-989e-8a6a83cee487)  

*Out of the distance participants moved, the highest average ratio goes to light activity. Sedentary distance will be 0 because there is usually 0 movement when one is sedentary.  

<ins> Avg. time sleeping and spent in bed </ins>  
```sql
SELECT ROUND(AVG(TotalMinutesAsleep)/60,2) AS Avg_Sleep_Hours,
ROUND(AVG(TotalTimeInBed)/60,2) AS Avg_Hours_in_Bed,
ROUND((ROUND(AVG(TotalTimeInBed)/60,2) - ROUND(AVG(TotalMinutesAsleep)/60,2)),2) AS Avg_Hours_in_Bed_Not_Sleeping
 FROM `bellabeat-case-study-410302.sleep.sleep_daily`
```
![Screenshot 2024-01-18 133438](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/30d437db-e176-4539-8b63-d89b937be0cf)  

*On avg, participants were sleeping for around 7 hours a day, spent a total of 7.64 hours in bed a day, which means around 40 minutes of staying in bed but not sleeping.  

<ins> Correlation Analysis </ins>  
Using Excel’s data analysis toolpak, let’s perform a correlation analysis between some variables.  

Total Steps against Calories:  
![Screenshot 2024-01-18 133531](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/8259af2c-dafc-451f-bba9-25346731fb79)  

Active Distance against Calories:  
![Screenshot 2024-01-18 133554](https://github.com/chanolivia/bellabeat_sql_case_study/assets/143843732/9f2451ef-80a0-4098-aa32-39b219bc7496)  







  












