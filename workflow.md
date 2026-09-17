# Taiwan Weather Forecast — Project Workflow

## 1. Project Overview

**Project Name:** Taiwan Weather Forecast Dashboard  
**Repository:** `timko1018/0916`  
**Main Goal:** Build a complete Taiwan weather-data application that retrieves public weather forecast data from the Central Weather Administration (CWA), parses and transforms the JSON response, stores structured data in SQLite, and presents an interactive dashboard with Streamlit.

The project is designed as an end-to-end practice project covering:

- REST API usage
- JSON parsing
- Python data processing
- Pandas
- SQLite and SQL
- Streamlit web application development
- Data visualization
- Taiwan map visualization
- Git/GitHub version control
- Streamlit Cloud deployment and automatic redeployment

---

## 2. Target Architecture

```text
                    Central Weather Administration
                              CWA Open Data
                                   │
                                   │ REST API
                                   ▼
                           ┌───────────────┐
                           │  Python       │
                           │  requests     │
                           └───────┬───────┘
                                   │
                                   ▼
                           ┌───────────────┐
                           │   JSON        │
                           │   Parsing     │
                           └───────┬───────┘
                                   │
                                   ▼
                           ┌───────────────┐
                           │ Data Cleaning  │
                           │ Pandas         │
                           └───────┬───────┘
                                   │
                                   ▼
                           ┌───────────────┐
                           │    SQLite     │
                           │    data.db    │
                           └───────┬───────┘
                                   │
                                   │ SQL Query
                                   ▼
                           ┌───────────────┐
                           │   Streamlit   │
                           │   Dashboard   │
                           └───────┬───────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
                 Charts          Tables          Map
                    │              │              │
                    └──────────────┴──────────────┘
                                   │
                                   ▼
                         Streamlit Community Cloud
                                   ▲
                                   │
                         GitHub Auto Deployment
                                   │
                              Git Push
```

---

## 3. Development and Deployment Philosophy

The project follows this order:

```text
1. GitHub Repository
       ↓
2. Basic Streamlit App
       ↓
3. CWA API Connection
       ↓
4. JSON Parsing
       ↓
5. Data Processing
       ↓
6. SQLite Database
       ↓
7. SQL Queries
       ↓
8. Streamlit Dashboard
       ↓
9. Temperature Charts
       ↓
10. Taiwan Map Visualization
       ↓
11. GitHub Push
       ↓
12. Streamlit Cloud Auto Deploy
```

The application should be developed incrementally. Each stage should be tested before moving to the next stage.

---

# 4. Phase 1 — GitHub Repository Setup

The GitHub repository is the source of truth for the project code.

Repository:

`https://github.com/timko1018/0916`

Recommended project structure:

```text
0916/
│
├── app.py                  # Streamlit entry point
├── cwa_api.py              # CWA API request functions
├── data_processor.py       # JSON parsing and data transformation
├── database.py             # SQLite operations
├── requirements.txt        # Python dependencies
├── README.md               # Project documentation
├── workflow.md             # Development workflow
├── .gitignore              # Files excluded from Git
│
└── data/
    └── .gitkeep            # Keep the directory in Git
```

Do not commit API keys, passwords, tokens, virtual environments, or generated local database files unless there is a specific reason to do so.

---

# 5. Phase 2 — Basic Streamlit Application

First verify that GitHub → Streamlit deployment works before adding external APIs or databases.

Minimal target:

```python
import streamlit as st

st.title("🇹🇼 Taiwan Weather Forecast")
st.write("Taiwan Weather Dashboard")
```

Success criteria:

- The application runs locally.
- The code can be pushed to GitHub.
- Streamlit Cloud can deploy the repository.
- The deployed page loads successfully.

---

# 6. Phase 3 — CWA API Integration

## Data Source

Use the Central Weather Administration Open Data platform.

Initial forecast dataset:

`F-C0032-001`

The first implementation should retrieve the forecast JSON using Python `requests`.

Conceptual flow:

```text
Streamlit / Python
       ↓
requests.get()
       ↓
CWA REST API
       ↓
JSON response
       ↓
Python dictionary
```

Example design:

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

### Security requirement

The CWA API key must **not** be hard-coded into the source code.

For Streamlit Cloud, store the key in Streamlit Secrets and access it through:

```python
import streamlit as st

API_KEY = st.secrets["CWA_API_KEY"]
```

A local development environment can use a local secrets/configuration file that is excluded from Git.

---

# 7. Phase 4 — JSON Parsing

The raw CWA response should be treated as an external data structure. Do not immediately assume that every nested field exists.

The parser should identify and extract the required fields from the JSON response.

Primary fields:

- `locationName`
- `Wx` — weather phenomenon
- `MaxT` — maximum temperature
- `MinT` — minimum temperature
- `PoP` — probability of precipitation
- `CI` — comfort index, if required
- forecast start/end time

Target normalized structure:

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

The parser should convert the nested JSON into a flat list of records or a Pandas DataFrame.

Example conceptual result:

| regionName | dataDate | minT | maxT | weather | pop |
|---|---|---:|---:|---|---:|
| 臺北市 | 2026-09-16 | 25 | 32 | 多雲 | 30 |
| 新北市 | 2026-09-16 | 25 | 32 | 多雲 | 40 |
| 臺中市 | 2026-09-16 | 24 | 33 | 晴時多雲 | 20 |

---

# 8. Phase 5 — Data Processing

Use Pandas to clean and standardize the extracted data.

Tasks:

1. Convert temperature values to numeric values.
2. Convert precipitation probability to numeric values.
3. Normalize date/time fields.
4. Handle missing values.
5. Remove duplicate records when necessary.
6. Standardize region names.
7. Sort data by region and forecast time.
8. Validate the final schema.

Target DataFrame:

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

---

# 9. Phase 6 — SQLite Database

SQLite is used to demonstrate structured data storage and SQL querying.

Initial table design:

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

Useful queries include:

```sql
SELECT DISTINCT regionName
FROM TemperatureForecasts;
```

```sql
SELECT *
FROM TemperatureForecasts
WHERE regionName = ?;
```

```sql
SELECT MAX(maxT), MIN(minT)
FROM TemperatureForecasts;
```

```sql
SELECT *
FROM TemperatureForecasts
WHERE dataDate = ?;
```

### Deployment note

SQLite is a local file database. When deployed on Streamlit Community Cloud, the local filesystem should not be treated as permanent storage. Therefore, SQLite should initially be considered a local/cache/demo database. If persistent historical storage is required later, migrate the storage layer to a hosted database or another persistent data service.

---

# 10. Phase 7 — Streamlit Dashboard

The dashboard should provide the following interaction flow:

```text
User opens dashboard
       ↓
Select region
       ↓
Select date
       ↓
Query SQLite / fetch current data
       ↓
Display forecast information
```

Core UI components:

### Region selector

```python
region = st.selectbox(
    "選擇地區",
    regions
)
```

### Date selector

```python
date = st.date_input("選擇日期")
```

### Weather summary cards

Display:

- Maximum temperature
- Minimum temperature
- Weather condition
- Probability of precipitation

### Data table

Use Streamlit to display the normalized forecast records.

### Temperature chart

Display `MaxT` and `MinT` as time-series lines.

Conceptual chart:

```text
Temperature
 35 ┤             ● MaxT
 30 ┤      ●───────●────●
 25 ┤ ●────● MinT
 20 ┤
    └──────────────────────
       Date →
```

---

# 11. Phase 8 — Taiwan Map Visualization

The advanced dashboard should visualize weather information geographically.

User flow:

```text
Select date
     ↓
Load all Taiwan regions
     ↓
Match forecast data with geographic coordinates
     ↓
Display temperature/weather markers
```

Each marker can represent:

- Region
- Current/forecast temperature
- Minimum temperature
- Maximum temperature
- Weather condition
- Precipitation probability

Example concept:

```text
          北部
        🟢 28°C

      桃園 🟢
              新竹 🟡

          臺中 🟠
           32°C

       嘉義 🟠

          高雄 🔴
           35°C
```

The map implementation can be added after the core API/database/dashboard pipeline is stable.

---

# 12. Recommended Application Architecture

Keep responsibilities separated instead of putting all code inside `app.py`.

## `app.py`

Responsible for:

- Streamlit UI
- User interaction
- Calling service functions
- Displaying charts/tables/maps

## `cwa_api.py`

Responsible for:

- CWA API URL
- HTTP requests
- Authentication
- API response validation

## `data_processor.py`

Responsible for:

- JSON parsing
- Data extraction
- Data cleaning
- DataFrame creation

## `database.py`

Responsible for:

- SQLite connection
- Table creation
- Insert/update operations
- SQL queries

This separation makes the project easier to debug, test, maintain, and extend.

---

# 13. GitHub → Streamlit Auto Deployment Workflow

The final deployment pipeline is:

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
Streamlit Cloud detects repository update
        ↓
Dependencies installed from requirements.txt
        ↓
Streamlit application restarted
        ↓
New version becomes available online
```

The GitHub repository should therefore remain the central source of the application's code.

---

# 14. `requirements.txt`

The initial dependency list should contain only packages actually used by the project.

Expected packages may include:

```text
streamlit
requests
pandas
```

Additional packages should be added only when required by the map or visualization implementation.

After changing dependencies, push the updated `requirements.txt` to GitHub so Streamlit Cloud can rebuild the environment.

---

# 15. Development Milestones

## Milestone 1 — Repository

- [ ] GitHub repository ready
- [ ] Project structure created
- [ ] `.gitignore` created
- [ ] `requirements.txt` created

## Milestone 2 — Streamlit

- [ ] Basic `app.py`
- [ ] Local Streamlit app works
- [ ] GitHub → Streamlit deployment works

## Milestone 3 — CWA API

- [ ] CWA API key configured
- [ ] API request succeeds
- [ ] JSON response received
- [ ] API errors handled

## Milestone 4 — Data Processing

- [ ] JSON structure understood
- [ ] Weather fields extracted
- [ ] Temperature values normalized
- [ ] Pandas DataFrame created

## Milestone 5 — SQLite

- [ ] Database created
- [ ] Table created
- [ ] Records inserted
- [ ] SQL queries verified

## Milestone 6 — Dashboard

- [ ] Region selector
- [ ] Date selector
- [ ] Weather summary
- [ ] Data table
- [ ] Temperature line chart

## Milestone 7 — Advanced Visualization

- [ ] Taiwan map
- [ ] Date-based map filtering
- [ ] Temperature visualization
- [ ] Weather/precipitation information

## Milestone 8 — Final Deployment

- [ ] Secrets configured
- [ ] GitHub repository cleaned
- [ ] No API keys committed
- [ ] Streamlit deployment verified
- [ ] README updated
- [ ] Final project URL recorded

---

# 16. Final User Experience

The final application should provide a simple workflow:

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

---

# 17. Future Extensions

After the basic system is complete, possible extensions include:

- Automatic periodic CWA data refresh
- Historical weather data storage
- Weather trend analysis
- Rain probability visualization
- Temperature anomaly analysis
- Weather alerts
- Additional CWA datasets
- Responsive mobile UI
- More detailed Taiwan map layers
- Data export to CSV
- GitHub Actions for testing/linting

AI-based analysis can also be added later, but it should be treated as an extension after the core data pipeline is stable.

---

# 18. Final Project Goal

The final project is not simply a weather webpage. It is an end-to-end data application demonstrating the following pipeline:

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

The completed project should demonstrate the ability to take real-world public data and turn it into a usable, maintainable, and deployable web application.
