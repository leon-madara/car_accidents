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
Solution:

Converted the date format in the CSV file to match the required format before importing.
Alternatively, use the following SQL command to update the date format after importing:
