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
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailySteps`.id) ) AS Total_Steps_ID

FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 

FULL OUTER JOIN `portfolio-363100.Bellabeat_Case_Study.dailySleep` ON `portfolio-363100.Bellabeat_Case_Study.dailyActivity`.ID = `portfolio-363100.Bellabeat_Case_Study.dailySleep`.ID
AND  `portfolio-363100.Bellabeat_Case_Study.dailyActivity`.ActivityDate = `portfolio-363100.Bellabeat_Case_Study.dailySleep`.SleepDay

FULL OUTER JOIN `portfolio-363100.Bellabeat_Case_Study.dailySteps` ON `portfolio-363100.Bellabeat_Case_Study.dailySleep`.ID = `portfolio-363100.Bellabeat_Case_Study.dailySteps`.ID
AND  `portfolio-363100.Bellabeat_Case_Study.dailySleep`.SleepDay = `portfolio-363100.Bellabeat_Case_Study.dailySteps`.ActivityDay
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

#### dailySteps Table
```sql
SELECT  
`portfolio-363100.Bellabeat_Case_Study.dailySteps`.Id AS ID , 
COUNT(`portfolio-363100.Bellabeat_Case_Study.dailySteps`.Id) AS Total_Act_Day, 
COUNT(DISTINCT(`portfolio-363100.Bellabeat_Case_Study.dailySteps`.ActivityDay)) AS Total_Act_Day_DIST

FROM `portfolio-363100.Bellabeat_Case_Study.dailySteps` 
GROUP By `portfolio-363100.Bellabeat_Case_Study.dailySteps`.Id
HAVING Total_Act_Day != Total_Act_Day_DIST
```
No duplicate found

