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

However, to refine it, used a new code with the **ROW_NUMBER()** to filter the results. 
1. Get data on where total accidents are above 5,000
2. List the first three.




