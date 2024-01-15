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

## III. Process  
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
<img



