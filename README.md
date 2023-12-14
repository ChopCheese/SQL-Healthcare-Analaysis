# SQL Healthcare Analysis


### Project Overview

An Analysis using Sqlite3 on synthetic healthcare database. The aim is to seek an understanding of the data, making data-driven decision and EDA of the healthcare sector performance.


### Data sources

healthcare dataset: Primary dataset used for this analysis "healthcare_dataset.csv" file, each row having unique patient with their information when they visit the hospital.


### Tools

- SQLite - Data Cleaning, EDA
- Jupyter Notebook - Data presentation and reports
- Pandas - Data import, connecting to SQLite


### Data Cleaning and preparation

Initial stage of data preparation phase, we performed several task:
1. First importing data into jupyter notebook environment using pandas library

```python

import pandas as pd
import sqlite3

csv_path = 'C:\\Users\\healthcare_dataset.csv'
df = pd.read_csv(csv_path)

```

2. Connecting imported dataframe with SQLite and renaming to easier reference

```python

cnn = sqlite3.connect('healthcare_sqldb')
df.to_sql('healthcare_db', cnn)

%load_ext sql

%sql sqlite:///healthcare_sqldb

```

3. Pre-process by checking the structure of the data

```sql

PRAGMA table_info(healthcare_db);

```

4. Checking null-values and remove them

```sql

SELECT COUNT(*) null_total FROM healthcare_db
WHERE   Name IS NULL OR Age IS NULL
        OR Gender IS NULL
        OR `Blood Type` IS NULL
        OR `Medical Condition` IS NULL
        OR `Date of Admission` IS NULL
        OR Doctor IS NULL
        OR Hospital IS NULL
        OR `Insurance Provider` IS NULL
        OR `Billing Amount` IS NULL
        OR `Room Number` IS NULL
        OR `Admission Type` IS NULL
        OR `Discharge Date` IS NULL
        OR Medication IS NULL
        OR `Test Results` IS NULL;

```

5. Adding additional id column to idetify unique patients

```sql

ALTER TABLE healthcare_db
ADD COLUMN Name_id INT

UPDATE healthcare_db
SET Name_id = (SELECT COUNT(*) FROM healthcare_db db2 WHERE db2.Name = healthcare_db.Name)


```

6. Converting date column from text to date format

```sql

UPDATE healthcare_db 
SET `Date of Admission` = DATE(`Date of Admission`)

UPDATE healthcare_db 
SET `Discharge Date` = DATE(`Discharge Date`)


```


7. Adding new column, days spent for each patient by finding the difference between discharge date and date of admission

```sql

UPDATE healthcare_db
SET days_spent = (
    SELECT JULIANDAY(`Discharge Date`) - JULIANDAY(`Date of Admission`)
    FROM healthcare_db AS sq
    WHERE sq.Name_id = healthcare_db.Name_id
    LIMIT 1
)

```


### Exploratory Data Analysis

After preparing and Cleaning the data we can start exploring the data by these task:
1. Finding the mean, max, min values for Age, Billing Amount, days spent

```sql


SELECT
    'Age' AS Metric,
    AVG(Age) AS Mean,
    MIN(Age) AS Min,
    MAX(Age) AS Max
FROM
    healthcare_db

UNION

SELECT
    'Billing Amount' AS Metric,
    ROUND(AVG(`Billing Amount`), 2) AS Mean,
    ROUND(MIN(`Billing Amount`), 2) AS Min,
    ROUND(MAX(`Billing Amount`), 2) AS Max
FROM
    healthcare_db

UNION

SELECT
    'Days Spent' AS Metric,
    AVG(days_spent) AS Mean,
    MIN(days_spent) AS Min,
    MAX(days_spent) AS Max
FROM
    healthcare_db

```


2. Getting total patient for each gender

```sql

SELECT
(SELECT count(*) FROM healthcare_db where Gender = 'Male') as Male_count,
(SELECT count(*) FROM healthcare_db where Gender = 'Female') as Female_count

```

3. Gathering the distribution of Blood Type

```sql

SELECT `Blood Type`, count(*) as Type_total FROM healthcare_db
GROUP BY `Blood Type`
ORDER BY Type_total DESC

```

4. Grouping unique medical condition of patient and find their total
5. Checking the list of Doctors with most patients
6. Query the top 10 most visited hospitals
7. Finding the total of each insurance provider
8. Getting the total admission type
9. Checking Medicine that are poupular among patients
10. Gathering total for each test results
11. Finding which age group visited the hospitals the most and how much they spent both total and average

```sql

SELECT 
CASE 
    WHEN Age BETWEEN 10 AND 20 THEN '10-20'
    WHEN Age BETWEEN 21 AND 30 THEN '21-30'
    WHEN Age BETWEEN 31 AND 40 THEN '31-40'
    WHEN Age BETWEEN 41 AND 50 THEN '41-50'
    WHEN Age BETWEEN 51 AND 60 THEN '51-60'
    WHEN Age BETWEEN 61 AND 70 THEN '61-70'
    WHEN Age BETWEEN 71 AND 80 THEN '71-80'
    ELSE '81-90'
    END AS Age_group,
    count(*) as Total_patient,
    SUM(days_spent) as Total_dayspent,
    ROUND(AVG(days_spent),2) as Avg_dayspent,
    ROUND(SUM(`Billing Amount`),2) as Total_bill,
    ROUND(AVG(`Billing Amount`),2) as Avg_bill
    
FROM healthcare_db

GROUP BY Age_group 
ORDER BY Age_group ASC

```

### Results

From this EDA we can gather that
- They are 10,000 unique patients
- Mean, min and max findings
	- The mean age is 51.4, min age is 18, and max age is 85
	- The mean billing amount is 25,516.81, min is 1000.18, and max is 49,995.9
	- The mean day spent is 15.56, min is 1, and max is 30
- Male patient accounted for 4925 while female 5075
- Blood type of the patients are relatively the same with AB- having the highest at 1275 while A- the lowest at 1238
- Asthma and cancer is the leader in medical condition at 1708 and 1703 respetively
- Dr. Michael Johnson had the most patient attended at 7
- Smith PLC having total patient of 19
- Cigna is the most popular insurance provider at 2040 while medicare the lowest at 1925
- Urgent admission appeared the most in the findings at 3391
- Penicillin is the most in demand medication at 2079
- Abnormal result is the highest test result at 3456
- Age group from 71-89 is critical despite having the 2nd most total patient, 24216.0, highest average day spent in hospital, 15.93, and highest average bill, 25632.13

### Recommendations

- Focus on recovering critical patients by distributing them to avoid overloading
- Restocking the most in demand medicine to avoid short stock
- Doctors that does not have enough patient can rotate to attend abnormal patients to avoid them increasing even more
- Giving patient unique id to track their heatlh and easy to make appointments

### Limitations and challenges

- most hospital having similar names but could not deduce if those hospitals are the same company or different location
- Patient having similar names need to be categorized
- Most findings the number are not far apart from each other thus could not make a bigger decision to improve the service


### References

1. [Kaggle](https://www.kaggle.com/datasets/prasad22/healthcare-dataset)
