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
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 
