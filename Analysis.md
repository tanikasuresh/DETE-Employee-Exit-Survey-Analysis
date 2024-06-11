## Data Cleaning
In order to ensure that the data is fit for analysis, we need to remove any extra columns, rename the columns for easier handling, and ensure that all values in a column are standardized.

First, I dropped several columns that were not useful for the analysis
- An example of this is: 
```sql
ALTER TABLE Surveys
DROP COLUMN 'My say';
```
Second, we must rename the columns to not include any spaces for easier handling while writing queries
```sql
ALTER TABLE Surveys
RENAME COLUMN 'Job Dissatisfaction' to jobDissatisfaction;
```
Furthermore, the original dataset uses 'Not Stated' to indicate missing values so, we need to represent those as nulls in each database column
- An example of this is:
```sql
UPDATE Surveys
SET noneOfTheAbove = NULL
where noneOfTheAbove = 'Not Stated';
```
Also, the ceaseDate column includes dates in the 'MM/YYYY' and 'YYYY' format which we need to standardize as 'YYYY' as the month is not necessary for the analysis and the 'YYYY' format is easier for calculations
- Using the substr function, we can single out the 'YYYY' portion of all the values
```sql
UPDATE Surveys
SET ceaseDate = substr(ceasedate,-4,4);
```
Next, we are focusing our analysis on the respondents who resigned from the company, indicated by 'Resignation- Other reasons', 'Resignation- Other employer', or 'Resignation- Move overseas/interstate'. In order to include all the data in these three categories, we must join them under one category
- This can be done by using 'like' and '%' to capture all three values whose strings start with 'Resignation'
```sql
UPDATE Surveys
SET SeparationType = 'Resignation'
where SeparationType like 'Resignation%';
```
Most importantly, The exit survey allows the respondent to choose one or more reasons for leaving or choose 'None of the Above'. We can use a CASE statement within the UPDATE statement to add a column that returns whether the respondent was dissatisfied with the company prior to resigning.
- If the respondent indicated any one of the following reasons, we can mark them as TRUE (dissatisfied)
  - interpersonalConflicts
  - jobDissatisfaction
  - departmentDissatisfaction
  - workEnvironment
  - lackOfRecognition
  - lackOfJobSecurity
  - workLocation
  - employmentConditions
  - workLifeBalance
  - Workload
- If the respondent indicated 'none of the above', we can mark them as null
- If the respondent indicated any of the remaining reasons, we can mark them as FALSE (not dissatisfied)
```sql
UPDATE Surveys
SET dissatisfaction = CASE
when interpersonalConflicts = 'True' or jobDissatisfaction = 'TRUE' or departmentDissatisfaction = 'TRUE' or workEnvironment = 'TRUE' or lackOfRecognition = 'TRUE' or lackOfJobSecurity = 'TRUE' or workLocation = 'TRUE' or employmentConditions = 'TRUE' or workLifeBalance = 'TRUE' or Workload = 'TRUE' then TRUE
when noneOfTheAbove = 'TRUE' then null 
else FALSE
```
### With these changes, our dataset is much more relevant and easier to manipulate for the analysis.

## Analysis

### Percentage of Each Separation Type 
```sql
SELECT SeparationType, count(*) AS num_of_responses FROM 
Surveys
group by SeparationType
ORDER BY num_of_responses DESC;
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/2.png)

Here, we see that resignations did seem to be the leading reason for exiting the company between 2012 and 2014 closely followed by age retirement. It is definitely worth analyzing the trends corresponding to Resignations to understand the problem and prevent the company from losing talent.

### Number of Resignations that were Dissatisfied

```sql
select SeparationType, dissatisfaction, count(*) as numOfResponses from Surveys
where SeparationType = 'Resignation'
group by dissatisfaction
order by numOfResponses desc
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/3.png)

Among the respondents who resigned from the company between 2012 and 2014, it seems that slightly more resigned due to some sort of dissatisfaction with the company.

### Number of responses indicating dissatisfaction based on Age
```sql
SELECT AGE, DISSATISFACTION, COUNT(*) AS numOfResponses FROM surveys
WHERE AGE IS NOT NULL and SeparationType = 'Resignation'
GROUP BY AGE, DISSATISFACTION
ORDER BY AGE ASC, dissatisfaction desc
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/5.png)
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/41.png)

In both cases, younger and older employees are resigning due to dissatisfaction with the company and there does not seem to be skewed towards a higher or lower age bracket

### Number of responses indicating dissatisfaction based on Position
```sql
select separationtype, position, count(*) as num_of_responses from Surveys
where separationtype = 'Resignation'
group by position
order by num_of_responses desc
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/6.png)

<img src="https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/Position%20%25%20Pie%20Chart.png" width = "300" height = "300"> 
<img src = "https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/color%20legend.png" width = "500" height "500">

Out of all the 311 resignations in DETE, teachers are responsible for over 40% of the resignations, which is far greater than other positions.

![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/Teacher%20Dissatisfaction.png)

<img src="https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/%25%20of%20Total%20Position%20Dissatisfaction.png" width = "600" height = "600"> 

### Number of responses indicating dissatisfaction based on Gender
```sql
select SeparationType, gender, count(*) as num_of_responses from Surveys
where SeparationType = 'Resignation'
group by gender
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/7.png)

It is clear that the number of females have resigned is more than 3x the number of males who have resigned from 2012 to 2014.

```sql
SELECT gender, DISSATISFACTION, COUNT(*) AS numOfResponses FROM Surveys
where gender is not null and SeparationType = 'Resignation'
GROUP BY gender, DISSATISFACTION
ORDER BY gender ASC, dissatisfaction desc
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/8.png)

Furthermore, approximately more than 50% of females who have resigned felt some dissatisfaction with the company.

### Number of responses indicating dissatisfaction based on Years of Service
```sql
select SeparationType, ceasedate-startdate as years, count(*) as num_of_responses from Surveys
where SeparationType = 'Resignation' and years is not null
group by years
order by num_of_responses desc
```

![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/9.png)

It seems that the most number of resignations from 2012 to 2014 occured by people who had been working there for only 5 years.
Interestingly enough, the top nine number of resignations all comprise of people who had been working there for less than ten years. 
Conversally,the number of resignations for people that have worked 10 or more years was only in the single digits, with an increase in the number of resignations as the years decrease.
This may be due to company loyalty, people who have worked there less do not feel a sense of loyalty to stay at the company.

<img src="https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/%25%20of%20Total%20-%20Years%20of%20Service.png" width = "600" height = "600"> 
