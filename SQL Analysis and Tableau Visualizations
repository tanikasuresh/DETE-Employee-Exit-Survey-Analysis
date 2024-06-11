## Data Cleaning
- In order to ensure that the data is fit for analysis, we need to remove any extra columns, rename the columns for easier handling, and ensure that all values in a column are standardized.

- First, I dropped several columns that were not useful for the analysis
- An example of this is: 
```sql
ALTER TABLE Surveys
DROP COLUMN 'My say';
```
- Second, we must rename columns so they do not include any spaces so the SQL code can handle it easier
```sql
ALTER TABLE Surveys
RENAME COLUMN 'Job Dissatisfaction' to jobDissatisfaction;
```
- Furthermore, the data uses 'Not Stated' to indicate missing values
- so, we need to represent those as nulls in the database
- An example of this is:
```sql
UPDATE Surveys
SET noneOfTheAbove = NULL
where noneOfTheAbove = 'Not Stated';
```
- Also, the ceaseDate column includes dates in the 'MM/YYYY' and 'YYYY' format which we need to standardize as 'YYYY' to better handle them during the analysis
```sql
UPDATE Surveys
SET ceaseDate = substr(ceasedate,-4,4)
```
- Using the substr function, we can single out the 'YYYY' portion of all the values

- Lastly, we are focusing our analysis on the responses who resigned from the company, indicated by 'Resignation- Other reasons', 'Resignation- Other employer', or 'Resignation- Move overseas/interstate' 
- In order to include all data in these three categories, we must join them under one category
```sql
UPDATE Surveys
SET SeparationType = 'Resignation'
where SeparationType like 'Resignation%'
```
- This can be done by using 'like' and '%' to capture all three values whose strings start with 'Resignation'

- With these changes, our dataset is much more relevant and easier to manipulate for our analysis.
