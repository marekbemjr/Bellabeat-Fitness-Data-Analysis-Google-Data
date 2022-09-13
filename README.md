# Bellabeat Fitness Data Analysis Google Data


_In order to answer the key business questions, I will follow the 6 steps of the data analysis process:_

### [Ask](#1-ask)
### [Prepare](#2-prepare)
### [Process](#3-process)
### [Analyze](#4-analyze)
### [Share](#5-share) 
### [Act](#6-act)

## Scenerio 
Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market.
Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. Your team have been asked to analyze smart device data to gain insight into how consumers are using their smart devices. The insights you discover will then help guide marketing strategy for the company.

## About the company
Urška Sršen and Sando Mur founded Bellabeat, a high-tech company that manufactures health-focused smart products.
Sršen used her background as an artist to develop beautifully designed technology that informs and inspires women around
the world. Collecting data on activity, sleep, stress, and reproductive health has allowed Bellabeat to empower women with
knowledge about their own health and habits. Since it was founded in 2013, Bellabeat has grown rapidly and quickly
positioned itself as a tech-driven wellness company for women.
By 2016, Bellabeat had opened offices around the world and launched multiple products (Bellabeat app, Leaf, Time, Spring). Bellabeat products became available
through a growing number of online retailers in addition to their own e-commerce channel on their website. The company
has invested in traditional advertising media, such as radio, out-of-home billboards, print, and television, but focuses on digital
marketing extensively. Bellabeat invests year-round in Google Search, maintaining active Facebook and Instagram pages, and
consistently engages consumers on Twitter. Additionally, Bellabeat runs video ads on Youtube and display ads on the Google
Display Network to support campaigns around key marketing dates. 


## 1. ASK 
**Business Taks: Analyze the available data fron Fitbit to gain insights and help guide marketing strategy for Bellabeat company to become a larger player in the global smart device market.**

**Primary stakeholders: Urška Sršen and Sando Mur, executive team members.**

**Secondary stakeholders: Bellabeat marketing analytics team.**

## 2. PREPARE
Data Source: https://www.kaggle.com/datasets/arashnic/fitbit

This Kaggle data set contains personal fitness tracker from 30 fitbit users. Fitbit users consented to the submission of
personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. It includes
information about daily activity, steps, and heart rate that can be used to explore users’ habits

The dataset includes 18 .csv files in long format. 

The data also follow a ROCCC approach:
- **Reliability**: The data is from 30 FitBit users who consented to the submission of personal tracker data.
- **Original**: The data is from 30 FitBit users who consented to the submission of personal tracker data.
- **Comprehensive**: Data minute-level output for physical activity, heart rate, and sleep monitoring. While the data tracks many factors in the user activity and sleep, but the sample size is small and data is recorded only during less then  2 months.  
- **Current**: Data is from April 2020 to May 2020. Data is not actual so the users habit may be different now. 
- **Cited**: Unknown

⛔ The dataset has limitations:
- The database contains only 30 users. This is a small sample, to be analysed on a larger sample and more recent is desirable for in-depth analysis.
- Upon further investigation with ```distinct()``` to check for unique user Id, the set shows 33 user data from daily_activity database, 24 from daily_sleep and only 8 from weight. There are 3 extra users and some users did not record their data for tracking daily activity and sleep. 

## 3. PROCESS

At this stage of the analysis, I decided to use R. 

After installing the packages 
```
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')
install.packages('skimr')
```
```
library(tidyverse) #wrangle data
library(janitor) #cleaning data
library(lubridate) #wrangle data attributes
library(skimr) #summary data
library(ggplot2) #visualize data
```
and loaded the selected databases to analyze
```
daily_activity <- read.csv("dailyActivity_merged.csv")
daily_sleep <- read.csv("sleepDay_merged.csv")
weight <- read.csv("weightLogInfo_merged.csv")
```
Examine the date, inspect data to see if there are any errors with formatting

```
head(daily_activity)
head(daily_sleep)
head(weight)

str(daily_activity)
str(daily_sleep)
str(weight)
```

Check for NA, and remove duplicates for three main tables: daily_activity, daily_slepp and weight:
```
sum(is.na(daily_activity))
sum(is.na(daily_sleep))
sum(is.na(weight))

sum(duplicated(daily_actdsleep_day))
sum(duplicated(daily_sleep))
sum(duplicated(weight))

sleep_day <- sleep_day[!duplicated(sleep_day), ]
```
After a view of the output, there were a few issues, action to do:
1. Removing weight$fat and daily_activity$logged_activities_distance, as it has little to no context and would not be helpful during the analysis 
2. Add extra column: a day of the week, sedentary hours & total active hours column for further analysis in daily_activity database.
3. daily_activity$ActivityDate — format CHR not as a date format
4. daily_sleep$SleepDay — format CHR not as a date format
5. weight_log$Date — format CHR not as a date format
6. the naming of the column names (camelCase)
7. Remove rows in which the total_active_hours & calories burned are 0

**Removing weight$fat and daily_activity$logged_activities_distance, as it has little to no context and would not be helpful during the analysis**

```
weight <- weight %>% 
  select(-c(fat))
daily_activity <-daily_activity %>% 
  select(-c(logged_activities_distance))
```
**Add extra column**

```
daily_activity$total_active_hours = round((daily_activity$very_active_minutes + daily_activity$fairly_active_minutes + daily_activity$lightly_active_minutes)/60, digits = 2)
daily_activity$sedentary_hours = round((daily_activity$sedentary_minutes)/60, digits = 2)

daily_sleep$hours_in_bed = round((daily_sleep$total_time_in_bed)/60, digits = 2)
daily_sleep$hours_asleep = round((daily_sleep$total_minutes_asleep)/60, digits = 2)
daily_sleep$time_taken_to_sleep = (daily_sleep$total_time_in_bed - daily_sleep$total_minutes_asleep)

 ```
 
**Change date format using as.Date() & as.POSIXct()**
```
daily_activity$activity_date <- as.Date(daily_activity$activity_date,'%m/%d/%y')
daily_sleep$sleep_day <- as.Date(daily_sleep$sleep_day, '%m/%d/%y')
weight$date <- parse_date_time(weight$date, '%m/%d/%y %H:%M:%S %p')
```

**Clean the column naming** 
```
daily_activity <- clean_names(daily_activity)
daily_sleep <- clean_names(daily_sleep)
weight <- clean_names(weight)
```
**Remove rows in which the total_active_hours & calories burned are 0.**
```
daily_activity <- daily_activity[!(daily_activity$calories<=0),]
daily_activity <- daily_activity_cleaned[!(daily_activity_cleaned$total_active_hours<=0.00),]
```

Export dateset to use in Tableau for visualization
```
write.csv(daily_activity, file ='fitbit_daily_activity.csv')
write.csv(daily_sleep, file = 'fitbit_sleep_log.csv')
write.csv(weight, file = 'fitbit_weight.csv')
```

## 4. ANALYZE

**Summary:**

Average steps taken, sedentary hours, very active minutes & total hours of sleep.
```
summary(daily_activity$total_steps)
summary(daily_activity$sedentary_hours)
summary(daily_activity$very_active_minutes)
summary(daily_sleep$hours_asleep)
```
![summary](https://user-images.githubusercontent.com/110094376/189719187-325a6463-acfb-4e00-bf41-55549463f386.png)

**The conclusions are:**

The average number of steps per day were 8319, which is within the 6000–8000 recommended steps per day.

The average sedentary hours were 15.87 hours, which is absurdly high, recommended limit of 7–10 hours.

The average very active minutes also is less of the recommended 30 minutes of vigorous exercise every day.

The average hours spent asleep (6.9) also barely hits the quota of the recommended sleep time of 7–9 hours.



### By using ggplot for this section of the analysis phase I checked which days are users most active:
```
ggplot(data = daily_activity) +
  aes(x = day_of_week, y = total_active_hours) +
  geom_col(fill =  'red') +
  labs(x = 'Day of week', y = 'Total very active minutes', title = 'Total activity in a week')
```
![total_active](https://user-images.githubusercontent.com/110094376/189724467-800985ef-e04b-4a73-b60c-1149a3679b6c.png)


```
ggplot(data = daily_activity) +
  aes(x = day_of_week, y = calories) +
  geom_col(fill =  'green') +
  labs(x = 'Day of week', y = 'Calories burned', title = 'Total calories burned in a week')
```
![image](https://user-images.githubusercontent.com/110094376/189724893-2eba6849-89e9-4b58-b941-886e829ac4bd.png)


```
ggplot(data = daily_activity) +
  aes(x = day_of_week, y = total_distance) +
  geom_col(fill =  'orange') +
  labs(x = 'Day of week', y = 'Total distance', title = 'Total distance taken in a week')
```
![image](https://user-images.githubusercontent.com/110094376/189725118-4c613929-913a-4415-bcf5-91882c320ca9.png)


Users spend more time engaged in physical activity specifically on Sundays, which then proceeds to wane throughout the week with a slight peak on Thursdays which then sees a slow climb on Saturdays.

### Next investigate the relationship between total active hours, total distance, and sedentary hours against calories burned:

```
ggplot(data = daily_activity) +
  aes(x= total_active_hours, y = calories) +
  geom_point(color = 'red') +
  geom_smooth() +
  labs(x = 'Total active hours', y = 'Calories burned', title = 'Calories burned vs active hours')
```
![image](https://user-images.githubusercontent.com/110094376/189726178-03102513-9292-4b98-aa8d-ca326f530c5a.png)


```
ggplot(data = daily_activity) +
  aes(x= total_distance, y = calories) +
  geom_point(color = 'orange')+
  geom_smooth() +
  labs(x = 'Total distance', y = 'Calories burned', title = 'Calories burned vs total distance')
```
![image](https://user-images.githubusercontent.com/110094376/189726431-5cdf002b-2999-4b9a-822f-a706f46b2d39.png)


```
ggplot(data = daily_activity) +
  aes(x= sedentary_hours, y = calories) +
  geom_point(color = 'purple') +
  geom_smooth(method = "loess") +
  labs(x = 'Sedentary hours', y = 'Calories burned', title = 'Calories burned vs sedentary hours')
```
![image](https://user-images.githubusercontent.com/110094376/189726701-2e6cd103-f068-49a7-9cce-3be6165f4bbb.png)

Positive correlation between calories burned and total distance/total active hours which indicates that the more time you spend engaged in physical activity, the more calories you tend to burn. The relationship between sedentary hours and calories burned after 17 hours the values drop, which may indicate fatigue and too much sedentary work.

### Next I would like to check relationship between weight & physical activity. To do that I marged two dataframes. 

_MERGE the tables so we can carry out plotting._

```
weight_merge <- merge(daily_activity, weight, by=c('id'))
```

### Analize and share visualization in Tableau

![image](https://user-images.githubusercontent.com/110094376/189907296-b00e85c5-5851-47ee-8c39-ee8ac9ef0a25.png)

![image](https://user-images.githubusercontent.com/110094376/189907581-19f85b26-fa5b-44ca-9559-41561c6cfc2c.png)


After we infer that users weighing around 60kg & 85kg are the most active. 



![image](https://user-images.githubusercontent.com/110094376/189908055-10b90bac-f177-48c3-86c4-5bd2de4a9926.png)

Confirmation that majorty of people will take a total of 5000-1000 steps during the week. However, it can also be seen that some users are below this value. 


![image](https://user-images.githubusercontent.com/110094376/189908848-892c8edf-771a-4f7c-98fa-a5953c85e05d.png)

![image](https://user-images.githubusercontent.com/110094376/189909558-07675cce-539d-496d-890b-a964c40ceccb.png)

Most users just simply spend too much time sedentary, mainly 10–21 hours and most users barely exercise as we can see in the huge spike in recorded counts near the 0-10 on the x-axis.


![image](https://user-images.githubusercontent.com/110094376/189910190-3a58982e-895d-4f99-8c9e-3d9715c31943.png)

The graph confirms the average sleep time, which almost amounts to a minimum sleep time of 7 hours. However, a certain group sleeps below this value. 


As we can see in the 2 graphs below, the most active users are between 55-60kg and 85-90kg. We also see a sharp increase in activity at 70kg and a decrease in activity  for users above 90kg (physical and numerical).

![image](https://user-images.githubusercontent.com/110094376/189913655-2f549512-d329-4cd9-a82f-4c85e8c48383.png)

![image](https://user-images.githubusercontent.com/110094376/189913840-47a5145d-7143-4f1e-aa81-4a754677bb74.png)


The last two graphs related with sleep and weight. Sleep deprivation is most prevalent in the 68-72 kg and 50-55 kg weight groups. 

![image](https://user-images.githubusercontent.com/110094376/189913995-31361994-4637-4b9d-8e07-e133df745f93.png)

![image](https://user-images.githubusercontent.com/110094376/189914129-aa2ec7a8-f2fd-45d7-b1ca-a1be790526a4.png)


