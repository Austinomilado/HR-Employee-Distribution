# HR-Employee-Distribution

### Project Overview

The HR Employee Dataset provides comprehensive insights into the workforce of an organization, encompassing various aspects such as employee demographics, performance metrics, turnover rate, and more. This overview aims to highlight key features and potential applications of the dataset.

### Tools

 . Excel - Data Cleaning
 
 . SQL - Data Analysis
 
 . Tableau - Creating Reports

### Data Cleaning/Preparation

In the initial data preparation phase, i performed the following tasks
1. Data loading
2. Handling missing values
3. Data cleaning and formatting
4. Imported Data from excel to sql for analysis

### Exploratory Data Analysis

EDA involves exploring the HR Employee Distribution Data to answer key question such as:

1. What is the gender breakdown of employees in the company?
2. What is the race/ethnicity breakdown of employees in the company?
3. What is the age distribution of employees in the company?
4. How many employees work at headquarters versus remote locations?
5. What is the average length of employment for employees who have been terminated?
6. How does the gender distribution vary across departments and job titles?
7. What is the distribution of job titles across the company?
8. Which department has the highest turnover rate?
9. What is the distribution of employees across locations by state?
10. How has the company's employee count changed over time based on hire and term dates?
11. What is the tenure distribution for each department?

### Data Cleaning and Formatting Process

#### Create a database called projects
CREATE DATABASE projects;

##### Make use of the database 
USE projects;

##### Name the dataset hr 
```sql
SELECT * 
FROM hr;
```

##### Changing the table Id to emp_id
```sql
ALTER TABLE hr
CHANGE COLUMN id emp_id VARCHAR(20) NULL;
```

##### Describe the data to know what it looks like
```sql
DESCRIBE hr;
```

##### Checking the birthdate data format to see if it is messy
```sql
SELECT birthdate 
FROM hr;
```

##### This line of code is setting the SQL mode sql_safe_updates to 0.
SET sql_safe_updates = 0;


##### Cleaning the birthdate using the update statement
```sql
UPDATE hr
SET birthdate = CASE
	WHEN birthdate LIKE '%/%' THEN date_format(str_to_date(birthdate, '%m/%d/%Y'), '%Y-%m-%d')
    WHEN birthdate LIKE '%-%' THEN date_format(str_to_date(birthdate, '%m-%d-%Y'), '%Y-%m-%d')
    ELSE NULL
END;
```


##### Changing the birthdate type format from text to date
```sql
ALTER TABLE hr
MODIFY COLUMN birthdate DATE;
```


##### Cleaning the birthdate using the update statement
```sql
UPDATE hr
SET hire_date = CASE
	WHEN hire_date LIKE '%/%' THEN date_format(str_to_date(hire_date, '%m/%d/%Y'), '%Y-%m-%d')
    WHEN hire_date LIKE '%-%' THEN date_format(str_to_date(hire_date, '%m-%d-%Y'), '%Y-%m-%d')
    ELSE NULL
END;
```


##### Changing the hire_date type format from text to date
```sql
ALTER TABLE hr
MODIFY COLUMN hire_date DATE;
```


##### Cleaning the termdate using the update statement
```sql
UPDATE hr
SET termdate = IF(termdate IS NOT NULL AND termdate !='', 
DATE(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC')), '0000-00-00')
WHERE true;
```


##### Display just the  termdate column to see what it looks like
```sql
SELECT termdate
FROM hr;
```

##### By setting the SQL mode to 'ALLOW_INVALID_DATES', you're essentially instructing the database to be more lenient when it comes to date-related operations.
SET sql_mode = 'ALLOW_INVALID_DATES';

##### Changing the termdate type format from text to date
```sql
ALTER TABLE hr
MODIFY COLUMN termdate DATE;
```


##### Adding a 'age' column to the dataset
```sql
ALTER TABLE hr 
ADD COLUMN age INT;
```


##### Updates the 'age' column in the 'hr' table with the calculated age of each employee based on their birthdate and the current date.
```sql
UPDATE hr
SET age = timestampdiff(YEAR, birthdate, CURDATE());
```


##### Return a result set containing the 'birthdate' and 'age' columns for all records in the 'hr' table.
```sql
SELECT birthdate,age 
FROM hr;
```


##### Return a result set containing two columns: 'youngest' and 'oldest', each with the respective youngest and oldest ages among all employees in the 'hr' table.
```sql
SELECT 
	min(age) AS youngest,
    max(age) AS oldest
FROM hr;
```


##### Return a single value representing the count of records in the 'hr' table where the age of employees is less than 21.
```sql
SELECT COUNT(*) 
FROM hr 
WHERE age < 21;
```


##### Return a single value representing the count of records in the 'hr' table where the termination date of employees is greater than current date.
```sql
SELECT COUNT(*)
FROM hr 
WHERE termdate > CURDATE();
```


##### Return a single value representing the count of records in the 'hr' table where the 'termdate' is '0000-00-00'
```sql
SELECT COUNT(*)
FROM hr
WHERE termdate = '0000-00-00';
```


##### Return a result set containing the 'location' column for all records in the 'hr' table
```sql
SELECT location 
FROM hr;
```

### QUESTIONS

##### 1. What is the gender breakdown of employees in the company?
```sql
SELECT gender, COUNT(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY gender;
```


##### 2. What is the race/ethnicity breakdown of employees in the company?
```sql
SELECT race, COUNT(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY race
ORDER BY count(*) DESC;
```


##### 3. What is the age distribution of employees in the company?
```sql
SELECT 
	min(age) AS youngest,
    max(age) AS oldest
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00';

SELECT
    CASE
      WHEN age >= 21 AND age <= 27 THEN '21-27'
      WHEN age >= 28 AND age <= 37 THEN '28-37'
      WHEN age >= 35 AND age <= 47 THEN '35-47'
      WHEN age >= 42 AND age <= 57 THEN '42-57'
      WHEN age >= 49 AND age <= 67 THEN '49-67'
      ELSE '68+'
    END AS age_group,
    count(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY age_group
ORDER BY age_group;

SELECT
    CASE
      WHEN age >= 21 AND age <= 27 THEN '21-27'
      WHEN age >= 28 AND age <= 37 THEN '28-37'
      WHEN age >= 35 AND age <= 47 THEN '35-47'
      WHEN age >= 42 AND age <= 57 THEN '42-57'
      WHEN age >= 49 AND age <= 67 THEN '49-67'
      ELSE '68+'
    END AS age_group, gender,
    count(*) AS count 
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY age_group, gender
ORDER BY age_group, gender;
 ```

  
##### 4. How many employees work at headquarters versus remote locations?
```sql
SELECT location, count(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY location;
```


##### 5. What is the average length of employment for employees who have been terminated?
```sql
SELECT 
    round(AVG(datediff(termdate, hire_date))/365,0) AS avg_length_employment
FROM hr
WHERE termdate <= curdate() AND termdate <> '0000-00-00' AND age >= 21;
```


##### 6. How does the gender distribution vary across departments and job titles?
```sql
SELECT department, gender, COUNT(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY department, gender
ORDER BY department;
```


##### 7. What is the distribution of job titles across the company?
```sql
SELECT jobtitle, COUNT(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY jobtitle
ORDER BY jobtitle DESC;
```


##### 8. Which department has the highest turnover rate?
```sql
SELECT department, 
       total_count,
       terminated_count,
       terminated_count/total_count AS termination_rate
FROM (
		SELECT department,
        count(*) AS total_count,
        SUM(CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate() THEN 1 ELSE 0 END) AS terminated_count
        FROM hr
        WHERE age >= 21
        GROUP BY department
        ) AS subquery
	ORDER BY termination_rate DESC;
```


##### 9. What is the distribution of employees across locations by city and state?
```sql
SELECT location_state, count(*) AS count
FROM hr
WHERE age >= 21 AND termdate = '0000-00-00'
GROUP BY location_state
ORDER BY count DESC;
```


##### 10. How has the company's employee count changed over time based on hire and term dates?
```sql
SELECT
    year,
    hires,
    terminations,
    hires - terminations AS net_change,
    round((hires - terminations)/hires * 100, 2) AS net_change_percent
FROM(
    SELECT 
		 YEAR(hire_date) AS year,
         count(*) AS hires,
         SUM(CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate() THEN 1 ELSE 0 END) AS terminations
         FROM hr
         WHERE age >= 21
         GROUP BY YEAR(hire_date)
         ) AS subquery
ORDER BY year ASC;
```

 
##### 11. What is the tenure distribution for each department?
```sql
SELECT department, round(AVG(datediff(termdate, hire_date))/365,0) AS avg_tenure
FROM hr
WHERE termdate <= curdate() AND termdate <> '0000-00-00' AND age >= 21
GROUP BY department;
```







