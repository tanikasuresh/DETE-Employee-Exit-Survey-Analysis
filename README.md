### The following data is from the Department of Education, Training and Employment (DETE) in Australia and comprises of employee exit surveys.

The data contains 51 columns and 822 rows that captures the background of the respondent and their opinion about the company prior to leaving. Our task will be to analyze the respondents that resigned from the company and better understand the trend behind their resignation from the company and whether there was any dissatisfaction with the company.

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
#### Now the dataset comprises of 8 columns:
- `separationType` - overall reason for departure 
- `ceaseDate` - year of departure
- `startDate` - year the respondent's employment began
- `years` - total years worked
- `position` - job role
- `dissatisfaction ` - whether the respondent was dissatisfied with the company
- `gender `
- `age` 
  
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

Here, we see that resignations did seem to be the leading reason for exiting the company between 2012 and 2014 closely followed by age retirement. It is definitely worth analyzing the trends corresponding to Resignations to understand the problem and prevent the company from losing talent in the future.

### Number of Resignations that were Dissatisfied

```sql
select SeparationType, dissatisfaction, count(*) as numOfResponses from Surveys
where SeparationType = 'Resignation'
group by dissatisfaction
order by numOfResponses desc
```
![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/SQL%20Images/3.png)

Among the respondents who resigned from the company between 2012 and 2014, it seems that slightly more did indeed resign due to some sort of dissatisfaction with the company.

![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/%25%20of%20Total%20-%20Resignations.png)

Furthermore, among those that resigned due to some dissatisfaction, `Work Life Balance`, `Job Dissatisfaction`, `Lack of Recognition`, `Department Dissatisfaction`, and `Workload Dissatisfaction` were the top five main reasons for resigning indicating that former employees were not happy with certain aspects of their jobs. 

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

Out of all the 311 resignations in DETE, teachers and teacher aides are responsible for over 60% of the resignations, which is greater than the 11 other positions combined.

![](https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/Teacher%20Dissatisfaction.png)

Specifically, for teachers who resigned due to dissatisfaction, `Work Life Balance`, `Job Dissatisfaction`, `Workload Dissatisfaction`, `Lack of Recognition`, and `Department Dissatisfaction` had the top five largest percentages of dissatisfaction. In other words, teachers who resigned mainly had an issue with these five categories.

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

<img src="https://github.com/tanikasuresh/DETE-Employee-Exit-Survey-Analysis/blob/main/Tableau%20Images/%25%20of%20Total%20-%20Years%20of%20Service.png" width = "600" height = "600"> 

The % of Total graph further indicates that for 0-10 years of employment there are more resignations split between dissatified and not, however, the greater the years of employment, the number of resignations decrease but those who do resign felt some sort of dissatisfaction with the company.

### Conclusion
There are several key insights based on this analysis
- Among employees who resigned most likely felt some dissatisfaction with `Work Life Balance`, `Job Dissatisfaction`, `Lack of Recognition`, `Department Dissatisfaction`, or `Workload Dissatisfaction`.
- Teachers and teacher aides are responsible for over 60% of the resignations
- The fewer the years of employment, the more resignations with a decrease in resignations as the years increase

Based on these takeaways, I would suggest a couple of changes moving forward. First, dig deeper into the areas that had higher percentages of dissatisfaction from former employees. For example, consider alleviating the work-life imbalance and workload stress that employees, specifically teachers may face. Furthermore, a lack of recognition could be fixed by being more attentive to employees, perhaps even employing some sort of Employee of the Month program to highlight employees that go above and beyond. Lastly, the trend of newer employees leaving more often, this may be due to a lack of company loyalty as people who have worked there fewer years do not feel a sense of committment towards the company. To combat this, consider company programs/events that create a sense of community among employees and transform the workplace into a more positive environment that employees are eager to work in. 
