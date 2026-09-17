# Taiwan Weather Forecast — Project Workflow

## 1. Project Overview

**Project Name:** Taiwan Weather Forecast Dashboard  
**Repository:** `timko1018/0916`  
**Goal:** Build an end-to-end weather data application using CWA Open Data, Python, JSON, Pandas, SQLite/SQL, Streamlit, visualization, GitHub, and Streamlit Cloud.

## 2. Target Architecture

```text
CWA Open Data
     ↓ REST API
Python / requests
     ↓
JSON Parsing
     ↓
Pandas Data Processing
     ↓
SQLite / SQL
     ↓
Streamlit Dashboard
     ├── Weather Summary
     ├── Data Table
     ├── Temperature Chart
     └── Taiwan Map
     ↓
GitHub
     ↓ Git Push
Streamlit Community Cloud
     ↓
Auto Deploy
```

## 3. Development Order

```text
1. GitHub Repository
2. Basic Streamlit App
3. CWA API Connection
4. JSON Parsing
5. Data Processing
6. SQLite Database
7. SQL Queries
8. Streamlit Dashboard
9. Temperature Charts
10. Taiwan Map Visualization
11. GitHub Push
12. Streamlit Cloud Auto Deploy
```

Each stage should be tested before moving to the next stage.

## 4. Recommended Project Structure

```text
0916/
├── app.py                  # Streamlit entry point
├── cwa_api.py              # CWA API requests
├── data_processor.py       # JSON parsing and data cleaning
├── database.py             # SQLite operations
├── requirements.txt        # Python dependencies
├── README.md               # Project documentation
├── .gitignore              # Ignored files
└── myplan/
    └── workflow.md        # Project workflow
```

Never commit API keys, passwords, tokens, virtual environments, or other secrets.

## 5. Phase 1 — Basic Streamlit

Verify the GitHub → Streamlit deployment path first.

```python
import streamlit as st

st.title("🇹🇼 Taiwan Weather Forecast")
st.write("Taiwan Weather Dashboard")
```

Success criteria:

- Local Streamlit app runs.
- Code is pushed to GitHub.
- Streamlit Cloud deploys the repository.
- Deployed page loads successfully.

## 6. Phase 2 — CWA API

Initial forecast dataset:

`F-C0032-001`

Use Python `requests` to retrieve JSON from the CWA REST API.

```python
import requests

url = "https://opendata.cwa.gov.tw/api/v1/rest/datastore/F-C0032-001"
params = {
    "Authorization": API_KEY,
    "format": "JSON"
}

response = requests.get(url, params=params)
response.raise_for_status()
data = response.json()
```

The API key must not be hard-coded. For Streamlit Cloud, use Streamlit Secrets:

```python
import streamlit as st
API_KEY = st.secrets["CWA_API_KEY"]
```

## 7. Phase 3 — JSON Parsing

Extract the important fields from the nested CWA response:

- `locationName`
- `Wx` — weather phenomenon
- `MaxT` — maximum temperature
- `MinT` — minimum temperature
- `PoP` — probability of precipitation
- `CI` — comfort index, if required
- forecast start/end time

Normalize the data into:

```text
regionName
forecastStart
forecastEnd
dataDate
minT
maxT
weather
pop
comfortIndex
```

The result should become a flat list or Pandas DataFrame.

## 8. Phase 4 — Data Processing

Use Pandas to:

1. Convert temperatures to numeric values.
2. Convert precipitation probability to numeric values.
3. Normalize date/time fields.
4. Handle missing values.
5. Remove duplicates when necessary.
6. Standardize region names.
7. Sort by region and forecast time.
8. Validate the schema.

## 9. Phase 5 — SQLite

Initial table:

```sql
CREATE TABLE IF NOT EXISTS TemperatureForecasts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    regionName TEXT NOT NULL,
    forecastStart TEXT,
    forecastEnd TEXT,
    dataDate TEXT NOT NULL,
    minT REAL,
    maxT REAL,
    weather TEXT,
    pop REAL,
    comfortIndex TEXT
);
```

Main operations:

```text
Create database
      ↓
Create table
      ↓
Insert forecast records
      ↓
Query records
      ↓
Return DataFrame
```

Useful queries:

```sql
SELECT DISTINCT regionName FROM TemperatureForecasts;
```

```sql
SELECT * FROM TemperatureForecasts WHERE regionName = ?;
```

```sql
SELECT MAX(maxT), MIN(minT) FROM TemperatureForecasts;
```

```sql
SELECT * FROM TemperatureForecasts WHERE dataDate = ?;
```

SQLite should initially be treated as a local/demo/cache database. Do not assume a local SQLite file provides permanent storage on Streamlit Cloud.

## 10. Phase 6 — Streamlit Dashboard

User flow:

```text
Open Dashboard
      ↓
Select Region
      ↓
Select Date
      ↓
Query Data
      ↓
Display Forecast
```

Core UI:

- Region selector
- Date selector
- Maximum temperature
- Minimum temperature
- Weather condition
- Precipitation probability
- Forecast data table
- MaxT/MinT time-series chart

## 11. Phase 7 — Taiwan Map

Advanced flow:

```text
Select Date
     ↓
Load Taiwan regions
     ↓
Match forecast data with geographic coordinates
     ↓
Display weather/temperature markers
```

Markers may display:

- Region
- Temperature
- MinT / MaxT
- Weather condition
- Precipitation probability

Implement the map after the API → processing → database → dashboard pipeline is stable.

## 12. Application Responsibilities

### `app.py`

- Streamlit UI
- User interaction
- Charts, tables, maps

### `cwa_api.py`

- CWA API URL
- HTTP requests
- Authentication
- Response validation

### `data_processor.py`

- JSON parsing
- Data extraction
- Data cleaning
- DataFrame creation

### `database.py`

- SQLite connection
- Table creation
- Insert/update operations
- SQL queries

Separating responsibilities keeps the project easier to debug, test, maintain, and extend.

## 13. GitHub → Streamlit Auto Deployment

```text
Developer changes code
        ↓
Local testing
        ↓
git add .
        ↓
git commit -m "..."
        ↓
git push
        ↓
GitHub main branch updated
        ↓
Streamlit Cloud detects update
        ↓
Install requirements.txt
        ↓
Restart Streamlit application
        ↓
Updated website
```

GitHub is the source of truth for the application code.

## 14. Requirements

Initial dependencies:

```text
streamlit
requests
pandas
```

Add map/visualization packages only when they are actually required.

## 15. Development Milestones

### Milestone 1 — Repository

- [ ] Repository ready
- [ ] Project structure created
- [ ] `.gitignore` created
- [ ] `requirements.txt` created

### Milestone 2 — Streamlit

- [ ] Basic `app.py`
- [ ] Local app works
- [ ] GitHub → Streamlit deployment works

### Milestone 3 — CWA API

- [ ] API key configured
- [ ] API request succeeds
- [ ] JSON response received
- [ ] API errors handled

### Milestone 4 — Data Processing

- [ ] JSON structure understood
- [ ] Weather fields extracted
- [ ] Temperature values normalized
- [ ] Pandas DataFrame created

### Milestone 5 — SQLite

- [ ] Database created
- [ ] Table created
- [ ] Records inserted
- [ ] SQL queries verified

### Milestone 6 — Dashboard

- [ ] Region selector
- [ ] Date selector
- [ ] Weather summary
- [ ] Data table
- [ ] Temperature line chart

### Milestone 7 — Advanced Visualization

- [ ] Taiwan map
- [ ] Date-based map filtering
- [ ] Temperature visualization
- [ ] Weather/precipitation information

### Milestone 8 — Final Deployment

- [ ] Secrets configured
- [ ] No API keys committed
- [ ] Streamlit deployment verified
- [ ] README updated
- [ ] Final project URL recorded

## 16. Final User Experience

```text
Open Taiwan Weather Forecast
             ↓
       Select a region
             ↓
        Select a date
             ↓
 ┌─────────────────────────┐
 │ Weather Summary         │
 │ Max Temperature         │
 │ Min Temperature         │
 │ Weather Condition       │
 │ Precipitation Probability│
 └─────────────────────────┘
             ↓
      Temperature Chart
             ↓
       Forecast Table
             ↓
       Taiwan Map
```

## 17. Future Extensions

- Automatic periodic CWA data refresh
- Historical weather data storage
- Weather trend analysis
- Rain probability visualization
- Temperature anomaly analysis
- Weather alerts
- Additional CWA datasets
- Responsive mobile UI
- More detailed Taiwan map layers
- CSV export
- GitHub Actions for testing/linting
- AI-based weather analysis after the core pipeline is stable

## 18. Final Goal

The finished project demonstrates an end-to-end data application:

```text
Public Open Data
      ↓
REST API
      ↓
Python
      ↓
JSON Parsing
      ↓
Data Cleaning
      ↓
Pandas
      ↓
SQLite / SQL
      ↓
Streamlit
      ↓
Interactive Visualization
      ↓
GitHub
      ↓
Streamlit Cloud
      ↓
Auto Deployment
```
