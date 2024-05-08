# HR-Employee-Distribution

### Project Overview

The HR Employee Dataset provides valuable insights into the organization's workforce composition, dynamics, and trends, facilitating informed decision-making across various HR functions, including recruitment, retention, talent development, and succession planning.

### Tools

 * Excel - Data Cleaning
 
 * SQL - Data Analysis
 
 * Tableau - Creating Reports

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

### HR Employee Report/Dashboard 

![Dashboard 1](https://github.com/Austinomilado/HR-Employee-Distribution/assets/86606293/3e24cd7d-618a-4344-bb7d-43317493f5b9)

### Data Used

#### Data - HR Data with over 22000 rows from the year 2000 to 2020.

#### Data Cleaning & Analysis - MySQL Workbench

#### Data Visualization - Tableau


### Summary of Findings

* There are more male employees
* White race is the most dominant while Native Hawaiian or other pacific islander are the least dominant.
* The youngest employee is 20 years old and the oldest is 57 years old
* 5 age groups were created (21-27, 28-37, 35-47, 42-57, 49-67). A large number of employees were between 28-37 followed by 42-57 while the smallest group was 49-67.
* A large number of employees work at the headquarters versus remotely.
* The average length of employment for terminated employees is around 8 years.
* The gender distribution across departments is fairly balanced but there are generally more male than female employees.
* The Engineering department has the highest turnover rate followed by Accounting. The least turn over rate are in the Legal and Auditing departments.
* A large number of employees come from the state of Ohio.
* The net change in employees has increased over the years, while the change in Termination has decreased over the years.
* The average tenure for each department is about 9 years with Sales and Services having the highest, While Human Resources and business Development having the lowest.


### Recommendation

Based on the insights derived from the HR Employee Dataset, several recommendations can be made to optimize HR strategies and enhance organizational effectiveness:

#### Diversity and Inclusion Initiatives:
* Implement initiatives to promote diversity and inclusion within the organization, particularly focusing on increasing representation of underrepresented groups such as Native Hawaiian or other Pacific Islander employees and females in certain departments.

#### Talent Acquisition and Retention:
* Develop targeted recruitment strategies to attract talent from a broader demographic pool, including regions beyond Ohio, to ensure a diverse workforce.
* Focus on retaining employees within the Engineering and Accounting departments by identifying and addressing factors contributing to turnover, such as career development opportunities and work-life balance initiatives.

#### Employee Engagement and Satisfaction:
* Enhance employee engagement efforts, particularly for remote workers, to ensure they feel connected and valued within the organization.
* Implement programs to improve overall employee satisfaction, taking into account factors such as compensation, career advancement opportunities, and workplace culture.

#### Succession Planning and Leadership Development:
* Develop robust succession planning programs to groom high-potential employees for leadership positions, considering the average tenure for each department and potential retirements in the future.
* Invest in leadership development programs, particularly in departments with high turnover rates, to strengthen leadership capabilities and reduce the impact of turnover.

#### Performance Management and Feedback:
* Enhance performance management processes to recognize and reward high-performing employees, thereby increasing employee motivation and engagement.
* Implement regular feedback mechanisms to understand employee concerns and address them proactively, contributing to a positive work environment and reducing turnover.

#### Data-Driven Decision-Making:
* Continuously monitor and analyze employee data to identify trends, patterns, and areas for improvement, enabling data-driven decision-making in HR strategies and initiatives.
* Utilize insights from the net change in employees over the years to forecast future workforce needs and align recruitment and retention efforts accordingly.

#### Employee Development and Training:
* Invest in training and development programs tailored to the needs of different age groups and departments, focusing on enhancing skills and competencies that are critical for organizational success.
* Provide opportunities for cross-functional training and career development to increase employee engagement and retention.

### Limitations

* Some records had negative ages and these were excluded during querying(967 records). Ages used were 21 years and above.
* Some termdates were far into the future and were not included in the analysis(1599 records). The only term dates used were those less than or equal to the current date.
