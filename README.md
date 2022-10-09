# Bellabeat-Case-Study
![image](https://user-images.githubusercontent.com/113477899/192513352-a094f40f-1a1c-45b7-846f-ea69c43f8443.png)

This is part of my Google Data Analytics Professional Certificate portfolio project,I will be imagine myself working for Bellabeat, a high-tech manufacturer of health-focused product for woman. 

## Business Task
* Non-Bellabeat users behavior using a smart device
* Provide more insight to the marketing team
* To help Bellabeat get more opportunities to grow
* Improving Bellabeat products


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
Id, COUNT(Id) AS Total_Act_Day, COUNT(DISTINCT(ActivityDate)) AS Total_Act_Day_DIST
FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 
GROUP By Id
HAVING Total_Act_Day != Total_Act_Day_DIST
```
No duplicate found

#### dailySleep Table
```sql
SELECT  
ID , 
COUNT(Id) AS Total_Act_Day, 
COUNT(DISTINCT(SleepDay)) AS Total_Act_Day_DIST
FROM `portfolio-363100.Bellabeat_Case_Study.dailySleep` 
GROUP By Id
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

#### Type of users and Active Level
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
Create a temporary table to count the percentage of each Active Level (total 33 users).  

```sql
WITH Percent_Active_Lvl AS (
SELECT  
ID, AVG(TotalSteps) AS Avg_Daily_Steps,AVG(Calories) AS Avg_Daily_Calories,
CASE
 WHEN AVG(TotalSteps) <5000 THEN 'Sedentary'
 WHEN AVG(TotalSteps) >=5000 AND AVG(TotalSteps) <7499 THEN 'Low active'
 WHEN AVG(TotalSteps) >=7500 AND AVG(TotalSteps) <9999 THEN 'Somewhat active'
 WHEN AVG(TotalSteps) >10000 THEN 'Active'
END AS Active_Lvl
FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity` 
GROUP BY Id )

SELECT
Active_Lvl, 
COUNT(Active_Lvl) AS Total_of_Active_Lvl, 
COUNT(Active_Lvl)/33*100 AS Percent_of_Active_Lvl
FROM Percent_Active_Lvl
GROUP BY Active_Lvl
```
![Sheet 1 (2)](https://user-images.githubusercontent.com/113477899/194024931-0afbdb5f-3c5e-40ac-8ed9-186a055f2dfa.png)

#### User most active day of week and total steps per day of week
```sql
WITH User_Most_Active_Day_Steps AS (SELECT ID, ActivityDate, TotalSteps, FORMAT_DATE('%A', ActivityDate) AS Day_Of_Week
FROM `portfolio-363100.Bellabeat_Case_Study.dailyActivity`)

SELECT  Day_Of_Week,AVG(TotalSteps) AS Total_DayofWeek_Steps,COUNT(Day_Of_Week) AS Total_Active_DayofWeek
FROM  User_Most_Active_Day_Steps
GROUP BY (Day_Of_Week)
ORDER BY case Day_Of_Week
WHEN 'Monday' THEN 1
WHEN 'Tuesday' THEN 2
WHEN 'Wednesday' THEN 3
WHEN 'Thursday' THEN 4
WHEN 'Friday' THEN 5
WHEN 'Saturday' THEN 6
WHEN 'Sunday' THEN 7
end 
```

#### User most sleep minutes day of week and total sleep minutes per day of week
```sql
WITH User_Most_Sleep_DayofWeek_SleepMinutes AS (SELECT ID, SleepDay, TotalMinutesAsleep, FORMAT_DATE('%A', SleepDay) AS Day_Of_Week
FROM `portfolio-363100.Bellabeat_Case_Study.dailySleep`)

SELECT  Day_Of_Week,AVG(TotalMinutesAsleep) AS Total_DayofWeek_SleepMinutes,COUNT(Day_Of_Week) AS Total_Sleep_DayofWeek
FROM  User_Most_Sleep_DayofWeek_SleepMinutes
GROUP BY (Day_Of_Week)
ORDER BY case Day_Of_Week
WHEN 'Monday' THEN 1
WHEN 'Tuesday' THEN 2
WHEN 'Wednesday' THEN 3
WHEN 'Thursday' THEN 4
WHEN 'Friday' THEN 5
WHEN 'Saturday' THEN 6
WHEN 'Sunday' THEN 7
end 
```
![Sheet 2 (2)](https://user-images.githubusercontent.com/113477899/194174915-46d0bb92-8adb-4a77-8298-86873e9b88ce.png)![Sheet 3](https://user-images.githubusercontent.com/113477899/194174959-9892bec1-f0e0-4a4b-b705-c63c41693ca9.png)

From the above graph, most days users walk the recommended number of steps except for Sunday. Another finding is that users didn’t sleep the recommended amount of time (8 hours a day).

#### Total hourly steps by hours

To find out the most active hour of the users.

```sql
SELECT  ID, ActivityDate, ActivityHour, AVG(StepTotal) AS AVG_Total_Steps
FROM  `portfolio-363100.Bellabeat_Case_Study.hourlySteps`
GROUP BY Id,ActivityDate, ActivityHour
```
![image](https://user-images.githubusercontent.com/113477899/194456518-9451483c-f37c-4b8d-9f4f-5ddf77edb96d.png)

The graph shows that the users are normally active between 8.00 am and 7.00 pm. The most peak period is during lunch time 12.00 pm to 2.00 pm and evening 5.00 pm to 7.00 pm.  

#### Use of the device
We would like to find out how often the user uses their device. The usage of the device can be categorised as follows:
*	High use– Users who use their device between 21 to 31 days
*	Moderate use – Users who use their device between 10 to 20 days
*	Low use – Users who use their device between 1 to 9 days


To find out the usage percentage (total 24 days)
```sql
WITH Days_Used AS (SELECT  ID, COUNT(SleepDay) AS Days_Used,
CASE
WHEN COUNT(SleepDay) >=1 AND COUNT(SleepDay) <10 THEN 'Low use'
WHEN COUNT(SleepDay) >=10 AND COUNT(SleepDay) <= 20 THEN 'Moderate use'
WHEN COUNT(SleepDay) >=21 THEN 'High use'
END AS Usage
FROM  `portfolio-363100.Bellabeat_Case_Study.dailySleep`
WHERE ID IS NOT NULL
GROUP BY ID)

SELECT Usage, COUNT(Usage)/24*100 AS Usage_Percent
FROM Days_Used
GROUP BY Usage
```
|Usage   	     |Usage_Percent	|
|--------------|--------------|
|High use      |50.0          | 
|Low use       |37.5	         |
|Moderate use  |12.5          |

![image](https://user-images.githubusercontent.com/113477899/194503294-15e7eafd-eaad-48cb-bd0d-e7fb8b9f9fbe.png)

## Conclusion
The result and trend that we have found from the analysis may help improve Bellabeat apps. The ideas:

__1.	Notification of steps__

From the analysis of the user’s average steps per day is about or more than 7,500 steps except for Sunday. An hourly notification updates the user about the steps they currently take to encourage them to move more to achieve their goal. 

__2.	Notification of the total daily sleep time__

Based on our results, the user’s daily average sleep minutes are lesser than the recommended sleep hour (8 hours/day). A notification that reminds the users 30-60mins before the pre-set sleep time would help the users to get ready themselves to go to bed. 

__3.	Interactive function on the apps__

An interactive function will help the users improve their lifestyle, motivate them to be more active and encourage them to sleep at least 8 hours a day. The function can compete the total daily step with another user and the winner will get a reward or appear on the leader board. Furthermore, [The National Sleep Foundation]( https://www.sclhealth.org/blog/2019/09/why-it-is-time-to-ditch-the-phone-before-bed/#:~:text=The%20National%20Sleep%20Foundation%20recommends,and%20start%20reading%20before%20bed.) recommends that people should stop using electronic devices, like phone, at least 30 minutes before bedtime. Notification that reminds users to put down their phones and get prepared to sleep.

With the datasets provided are small (30 users) and do not show the user demographic details. Knowing the user’s age and gender will help to find out more different trends about the user’s behaviour using a smart device.



