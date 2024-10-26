# Health Analytics Mini Case Study  

## 📋 Project Overview  
This project analyzes the **`health.user_logs`** dataset to provide key insights for **Health Co's General Manager of Analytics**. The goal is to assess user engagement by exploring the types and frequency of measurements logged by users, helping the board understand trends among the active user base. 

### Key Business Questions:
1. **How many unique users** exist in the dataset?  
2. **What is the average and median number of measurements per user?**  
3. **How many users logged 3+ and 1,000+ measurements?**  
4. **How many users logged blood glucose measurements?**  
5. **How many users logged at least 2 types of measurements?**  
6. **How many users recorded all three types**: blood glucose, weight, and blood pressure?  
7. **What is the median systolic and diastolic blood pressure** for users who recorded blood pressure?  

---

## Dataset Structure  
The **`health.user_logs`** dataset includes the following columns:  
- **`user_id`**: Unique identifier for users  
- **`measurement_type`**: Type of measurement (e.g., 'blood glucose', 'weight', 'blood pressure')  
- **`measurement_value`**: The value of the measurement  
- **`systolic`** / **`diastolic`**: Blood pressure readings (if applicable)  
- **`timestamp`**: Date and time of the measurement  

---

## 🛠️ SQL Queries  

### 1. **How many unique users exist in the dataset?**  
```sql
SELECT COUNT(DISTINCT user_id) AS unique_users
FROM health.user_logs;
```
> ### **Answer**
| COUNT  | 
| -------|
| 554    | 

---
### **For questions 2-8 we created a temporary table**
```sql
DROP TABLE IF EXISTS user_measure_count;

-- Create a temporary table with user measure counts
CREATE TEMP TABLE user_measure_count AS 
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) AS unique_measures
FROM health.user_logs
GROUP BY id;
```
### **Run temp table query first row**

```sql
SELECT *
FROM user_measure_count
LIMIT 3
```
> ### **Answer** 
| id       | measure_count          | unique_measures |
| ------------- |:-------------:| -----:|
| 004beb6711843b214e80d73df57a3680fdf9700a | 3 | 2 |
| 007fe1259a4283a991e1f2835ddcdedacf78dde9 | 1 | 1 |
| 008dd1dc1728bb0b420188963905d259c5533149 | 1 | 1 |
---
### 2. **How many total measurements do we have per user on average?**  

```sql
SELECT
 ROUND( AVG(measure_count)) AS average_value
 FROM user_measure_count
```
> ### **Answer**
| average_value | 
| -------|
| 79    | 

---
### 3. **What about the median number of measurements per user?**  

```sql
SELECT
 PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) as median_value
FROM 
user_measure_count
```
> ### **Answer**
| median_value | 
| -------|
| 2    | 

---
### 4. **How many users have 3 or more measurements?**  

```sql
SELECT 
COUNT(*) 
FROM user_measure_count
WHERE  measure_count >= 3
```
> ### **Answer**
| COUNT | 
| -------|
| 209    | 

---
### 5. **How many users have 1,000 or more measurements?**  

```sql

SELECT 
COUNT(*) 
FROM user_measure_count
WHERE measure_count >=1000
```
> ### **Answer**
| COUNT | 
| -------|
| 5   | 

---
### **Looking at the logs data - what is the number of the active user base who:**
---

### 6. **Have logged blood glucose measurements?**  

```sql

SELECT 
measure, 
COUNT (DISTINCT id ),
ROUND(100 *COUNT (DISTINCT id )::NUMERIC / SUM (COUNT (DISTINCT id )) OVER(),2) as percent
FROM health.user_logs
GROUP BY measure
LIMIT 1
```
> ### **Answer** 
| measure       | count          | percent |
| ------------- |:-------------:| -----:|
| blood_glucose | 325 | 40.22 |

---
### 7. **Have at least 2 types of measurements?**  

```sql

WITH two_withmeasures AS (
  SELECT *
  FROM user_measure_count 
  WHERE unique_measures >= 2
)

SELECT 
COUNT(DISTINCT id ) as user_with_two_measures,
ROUND(100 *COUNT (DISTINCT id )::NUMERIC / (SELECT COUNT (DISTINCT id ) FROM user_measure_count),2) as percent
FROM two_withmeasures 
```

> ### **Answer** 
| user_with_two_measures      | percent         |
| ------------- |:-------------:| 
| 204| 36.82 | 

---
### 8. **Have all 3 measures - blood glucose, weight and blood pressure?**  

```sql
WITH three_measures AS (
  SELECT *
  FROM user_measure_count 
  WHERE unique_measures = 3
)

SELECT 
COUNT(DISTINCT id ) as user_three_measures,
ROUND(100 *COUNT (DISTINCT id )::NUMERIC / (SELECT COUNT (DISTINCT id ) FROM user_measure_count),2) as percent
FROM three_measures 
```

> ### **Answer** 
| user_three_measures     | percent         |
| ------------- |:-------------:| 
| 50| 9.03 | 

---
### **For users that have blood pressure measurements:**
---

### 9. **What is the median systolic/diastolic blood pressure values?**  

```sql
SELECT
  'blood_pressure' AS measure_name,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS systolic_median,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS diastolic_median
FROM health.user_logs
WHERE measure = 'blood_pressure';
```
> ### **Answer** 
| measure_name      | systolic_median         | diastolic_median |
| ------------- |:-------------:| -----:|
| blood_pressure | 126 | 79 |














