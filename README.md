# car_accidents
This project performs Exploratory Data Analysis (EDA) on a vehicle dataset using PostgreSQL. It involves loading the data, cleaning missing values, and analyzing vehicle types, ages, and impact points to gain actionable insights for further analysis or model development.
## Steps for Setting Up and Analyzing Data

### Create a Database in pgAdmin

- Database Name: `road_accident`
- Description: A database to store and analyze road accident data.

```sql
-- No SQL code required, created via pgAdmin interface
```

Create a Table for Storing Accident Data
Table Name: car_acc
Schema Definition:

```
CREATE TABLE car_acc (
    AccidentIndex VARCHAR,
    Severity VARCHAR,
    Date DATE,
    Day VARCHAR,
    SpeedLimit INTEGER,
    LightConditions VARCHAR,
    WeatherConditions VARCHAR,
    RoadConditions VARCHAR,
    Area VARCHAR
);
```

Import Data into the Table
  -  There was an error due to wrong date format.
  -  Excel's *text to columns* could not fix the issue.

### Imported to Google Colab
I used the following lines of code

```python
import pandas as pd

df = pd.read_csv(r'/content/accident.csv')

df['Date'] = pd.to_datetime(df['Date'], format='%d/%m/%Y').dt.strftime('%Y/%m/%d')

df
```

Once I finished, I downloaded the file using the code

```python
from google.colab import files
df.to_csv('modified_accident.csv', index=False)
files.download('modified_accident.csv')
```

Original Date Format: DD/MM/YYYY
Required Date Format: YYYY/MM/DD

The newly downloaded file now meets the criteria, allowing upload to PostgreSQL.

### The second part included uploading a second vehicle's table

Create a Table for Storing Vehicle Data
Table Name: vehicleaccidents
Schema Definition:

```sql
CREATE TABLE VehicleAccidents (
    VehicleID VARCHAR(255) PRIMARY KEY,
    AccidentIndex VARCHAR(255),
    VehicleType VARCHAR(255),
    PointImpact VARCHAR(255),
    LeftHand VARCHAR,
    JourneyPurpose VARCHAR(255),
    Propulsion VARCHAR(255),
    AgeVehicle INTEGER
);
```
Although the *LeftHand* column is essentially boolean, I chose *varchar* because some values were variable characters and that removed complications when importing it to the database


## The Exploratory Data Analysis (EDA)
### Questions
1. What is the distribution of different vehicle types involved in accidents?
the SQL code used:

```sql

SELECT VehicleType, COUNT(*) AS count
FROM vehicleaccidents
GROUP BY VehicleType
ORDER BY count DESC;
```

![sql 1](https://github.com/leon-madara/car_accidents/assets/147078093/4756a729-90e5-4d03-b94f-2b1775c012ef)

The output shows the distribution of different vehicle types in descending order, the majority being cars, with a substantial margin.

2. How does the point of impact vary across different vehicle types?

```SQL
SELECT VehicleType, PointImpact, COUNT(*) AS count
FROM vehicleaccidents
GROUP BY VehicleType, PointImpact
ORDER BY VehicleType, count DESC;
```
The output included data for all vehicles. 

![point of impact](https://github.com/leon-madara/car_accidents/assets/147078093/79bb6aec-8f10-40d0-b8fc-55d8e9a14a4d)

However, to refine it, used a new code with the **RANK()** and **HAVING** CLAUSE to filter the results. 
 - Get data on where total accidents are above 1,000 to see the highest cases.

```sql
SELECT VehicleType, PointImpact, COUNT(*) AS count, 
    RANK() OVER(PARTITION BY VehicleType ORDER BY COUNT(*) DESC) as rank
FROM vehicleaccidents
GROUP BY VehicleType, PointImpact
HAVING COUNT(*) > 1000;
```
The results
![having1](https://github.com/leon-madara/car_accidents/assets/147078093/ab02c0a3-54e7-445f-a2d3-0ff8d2b569fe)

3. What is the average age of vehicles involved in accidents based on severity?

```sql
SELECT ROUND(AVG(agevehicle), 1) AS avg_acc_age, severity
FROM vehicleaccidents v
JOIN car_acc c ON c.accidentindex = v.accidentindex
GROUP BY severity;
```

output

![avg cc](https://github.com/leon-madara/car_accidents/assets/147078093/f599015c-ced6-4cd2-a77a-b9d76828a0fb)

Seems there is no significant impact of age on accident severity

4. How many vehicles are left-hand drive versus right-hand drive?

```sql
WITH cte AS (
    SELECT COUNT(lefthand) AS lefthand
    FROM vehicleaccidents
    WHERE lefthand = 'No'
),
cte2 AS (
    SELECT COUNT(lefthand) AS righthand
    FROM vehicleaccidents
    WHERE lefthand != 'No'
)
SELECT cte.lefthand, cte2.righthand
FROM cte, cte2;
```

I used a **Common Table Expression** (CTE)

Output

![left and righthand](https://github.com/leon-madara/car_accidents/assets/147078093/b10706a5-b266-4343-ad3e-c8b139377c32)

There is a considerable difference between lefthand and righthand cars invovled in accidents, perhaps because its a lefthand drive couontry.

5. What is the distribution of journey purposes for vehicles involved in accidents?

```sql
select  journeypurpose, count(journeypurpose) as count
from vehicleaccidents
group by journeypurpose
order by count(journeypurpose) desc;
```

Output

![journey purpose](https://github.com/leon-madara/car_accidents/assets/147078093/43d19146-efbf-4830-b91e-0e1d37191968)

Seems like majority of accidents are related to work activities

6. Which types of propulsion are most common in vehicles involved in accidents?

```sql
select  propulsion, count(propulsion) as count
from vehicleaccidents
group by propulsion
order by count(propulsion) desc;
```

Output

![propulsion](https://github.com/leon-madara/car_accidents/assets/147078093/0ce0ffae-825a-4bc2-94bd-fb3a784a951e)

Many accidents involve vehicles propelled by Petrol, Heavy oil, Undefined

![propulsion 2](https://github.com/leon-madara/car_accidents/assets/147078093/20a7fbef-f4b7-4716-bbc4-e5529397abb2)
Created using Excel

7. Correlation between day, accident, and severity?

## First, we start by doing the count of accident by day 

```sql
select  day, count(severity) as severity_type
from car_acc
group by day
order by count(severity) desc;
```

Output
![day acc](https://github.com/leon-madara/car_accidents/assets/147078093/c18a380d-bb85-4831-b578-7b5209db962b)

![acc day](https://github.com/leon-madara/car_accidents/assets/147078093/2e9cad83-67b6-4e79-a335-3554ff47f990)
Excel chart

From the results, Sunday has the least, and all days have accidents above 12,000 with Friday having the most.

## Correlation between severity of accidents and day

```sql
select day,
	count(case when severity = 'Slight' then 1 end) as Slight,
	count(case when severity = 'Serious' then 1 end) as Serious,
	count(case when severity = 'Fatal' then 1 end) as Fatal
from car_acc
where severity IN ('Slight', 'Serious', 'Fatal')
group by day
order by day;
```

![count day acc](https://github.com/leon-madara/car_accidents/assets/147078093/0b212714-fb5a-4bc2-b118-6358732de450)

![excel acc days type](https://github.com/leon-madara/car_accidents/assets/147078093/2f9c07eb-dc86-4c85-a6b0-c0224dcc80b1)
Excel

![conditional form](https://github.com/leon-madara/car_accidents/assets/147078093/b162417e-55e5-4f43-bb22-49613b6fc577)


###
Based on the data from the conditionally formatted cells from **Excel**, Friday had a consistent figure. However, Saturday had the highest fatal accidents, while acciident trend for Sunday was consistent for Slight and Serious except for fatal.

8. Correlation between roadconditions and accident severity

```sql
select roadconditions,
	count(case when severity = 'Slight' then 1 end) as Slight,
	count(case when severity = 'Serious' then 1 end) as Serious,
	count(case when severity = 'Fatal' then 1 end) as Fatal
from car_acc
where severity IN ('Slight', 'Serious', 'Fatal')
group by roadconditions
order by roadconditions;
```

Output

![roadconditions severity](https://github.com/leon-madara/car_accidents/assets/147078093/93fa2eba-8770-4418-912a-d5ebd7be6749)

The data shows that most accidents occur when the road is dry and wet or damp. However, They significantly occur during dry road conditions. 

9. Is there a correlation between rural and urban accidents and propulsion type?

```sql
select propulsion,
	count(case when area = 'Urban' then 1 end) as Urban,
	count(case when area = 'Rural' then 1 end) as Rural
from car_acc c
join vehicleaccidents va on c.accidentindex = va.accidentindex
where area IN ('Urban', 'Rural')
group by propulsion
order by Urban, Rural;
```

Output

![rural urban](https://github.com/leon-madara/car_accidents/assets/147078093/218ffa3c-60e5-45b4-a156-1cdbf0d571b1)

Results
- Most accidents both in Rural and Urban areas include Petrol, Heavy oil, undefined, and electric.
- Petrol is the most common and highest in both urban and rural areas.


