# Bellabeat-Case-Study
![image](https://user-images.githubusercontent.com/113477899/192513352-a094f40f-1a1c-45b7-846f-ea69c43f8443.png)

This is part of my Google Data Analytics Professional Certificate portfolio project,I will be imagine myself working for Bellabeat, a high-tech manufacturer of health-focused product for woman. 

## Business Task
* Non-Bellabeat consumer behavior using a smart device
* Bellabeat VS Npn-Bellabeat consumer behavior
* To help Bellabeat get more opportunities to grow
* Provide more insight to the marketing team

## Data Source
* Fitbit Fitness Tracker Data – Public Domain, through [Mobius](https://www.kaggle.com/arashnic/datasets) (user consented to provide their personal data)

## Preview and Cleaning Process 

### Checking number of users
```sql
SELECT  
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailyActivity`.id) )AS Total_Daily_Activity_ID,
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailySleep`.id) ) AS Total_Sleep_ID,
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.hourlySteps`.id) ) AS Total_Steps_ID

FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 

FULL OUTER JOIN `portfolio-363100.Bellabeat_Case_Study.dailySleep` ON `portfolio-363100.Bellabeat_Case_Study.dailyActivity`.ID = `portfolio-363100.Bellabeat_Case_Study.dailySleep`.ID
AND  `portfolio-363100.Bellabeat_Case_Study.dailyActivity`.ActivityDate = `portfolio-363100.Bellabeat_Case_Study.dailySleep`.SleepDay

FULL OUTER JOIN `portfolio-363100.Bellabeat_Case_Study.hourlySteps` ON `portfolio-363100.Bellabeat_Case_Study.dailySleep`.ID = `portfolio-363100.Bellabeat_Case_Study.hourlySteps`.ID
AND  `portfolio-363100.Bellabeat_Case_Study.dailySleep`.SleepDay = `portfolio-363100.Bellabeat_Case_Study.hourlySteps`.ActivityDate
```
|Total_Daily |Total_Sleep|Total_Steps
|------------|-----------|-----------|
|33          |24         |33         |

### Checking Duplicates

#### dailyActivity Table
```sql
SELECT  
`portfolio-363100.Bellabeat_Case_Study.dailyActivity`.Id AS ID , 
COUNT(`portfolio-363100.Bellabeat_Case_Study.dailyActivity`.Id) AS Total_Act_Day, 
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailyActivity`.ActivityDate)) AS Total_Act_Day_DIST

FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 
GROUP By `portfolio-363100.Bellabeat_Case_Study.dailyActivity`.Id
HAVING Total_Act_Day != Total_Act_Day_DIST
```
No duplicate found

#### dailySleep Table
```sql
SELECT  
`portfolio-363100.Bellabeat_Case_Study.dailySleep`.Id AS ID , 
COUNT(`portfolio-363100.Bellabeat_Case_Study.dailySleep`.Id) AS Total_Act_Day, 
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailySleep`.SleepDay)) AS Total_Act_Day_DIST

FROM `portfolio-363100.Bellabeat_Case_Study.dailySleep` 
GROUP By `portfolio-363100.Bellabeat_Case_Study.dailySleep`.Id
HAVING Total_Act_Day != Total_Act_Day_DIST
```
3 duplicates found
|ID   	    |Total_Act_Day	|Total_Act_Day_DIST	
|-----------|---------------|-------------------|
|4388161847 |24             |23                 |
|4702921684 |28             |27                 |
|8378563200 |32             |31                 |

#### hourlySteps Table
```sql
SELECT ID,
COUNT(DISTINCT(ActivityDate)) AS Total_Act_Day,
COUNT(DISTINCT(ActivityHour)) AS Total_Hr_A_Day
FROM `portfolio-363100.Bellabeat_Case_Study.hourlySteps`
GROUP BY ID 
HAVING Total_Hr_A_Day != 24
```
No duplicate found

## Analyze Phase

This analysis is to find trends and determine if they can help us on Bellabeat's marketing strategy.
According to [this article](https://www.10000steps.org.au/articles/healthy-lifestyles/counting-steps/), the daily amount of step and activity levels can be classified as below:
* Sedentary - less than 5,000 steps per day 
* Low active - 5,000 to 7,499 steps per day
* Somewhat active - 7,500 to 9,999 steps per day
* Active - more than 10,000 steps per day

Finding the users average of daily steps, daily calories and daily sleep in minutes
```sql
SELECT  
ID, AVG(TotalSteps) AS Avg_Daily_Steps,AVG(Calories) AS Avg_Daily_Calories,
FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 
GROUP BY Id
```
```sql
SELECT ID, AVG(TotalMinutesAsleep) AS Avg_Daily_Sleep_MM
FROM `portfolio-363100.Bellabeat_Case_Study.dailySleep` 
GROUP BY Id
```
Classify activity levels by daily average steps

```sql
SELECT  
ID, AVG(TotalSteps) AS Avg_Daily_Steps,AVG(Calories) AS Avg_Daily_Calories,
CASE
 WHEN AVG(TotalSteps) <5000 THEN 'Sedentary'
 WHEN AVG(TotalSteps) >=5000 AND AVG(TotalSteps) <7499 THEN 'Low active'
 WHEN AVG(TotalSteps) >=7500 AND AVG(TotalSteps) <9999 THEN 'Somewhat active'
 WHEN AVG(TotalSteps) >10000 THEN 'Active'
END AS Active_Lvl
FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 
GROUP BY Id
```
Create a temporary table to count the percentage of each Active Level.  
