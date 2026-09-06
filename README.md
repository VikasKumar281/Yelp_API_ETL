# 🍽️ Yelp API ETL — API Data to PostgreSQL

**A Python-based ETL pipeline that extracts business data from the Yelp Fusion API, transforms the API response into structured records, and loads the data into a PostgreSQL database.**

---

## 📌 Overview

This project demonstrates a practical **ETL (Extract, Transform, Load) pipeline** that fetches business information from the **Yelp Fusion API** and stores the processed data in **PostgreSQL**.

The application allows users to search for businesses using command-line parameters such as:

- 🔎 Search term
- 📍 Location
- 💰 Price level

The pipeline then:

```text
User Input
    │
    ▼
Yelp Fusion API
    │
    ▼
Extract Business Data
    │
    ▼
Parse & Transform JSON
    │
    ▼
PostgreSQL
    │
    ▼
yelp.business
```

The main objective is to demonstrate how external API data can be extracted, transformed, and persisted into a relational database using a modular Python architecture.

---

# 🚀 Key Features

- 🌐 **Yelp Fusion API integration**
- 🔎 Search businesses by term
- 📍 Location-based business search
- 💰 Price-level filtering
- 🔄 End-to-end ETL workflow
- 🧹 Nested JSON parsing and transformation
- 🗃️ PostgreSQL data loading
- 🧱 Automatic schema and table creation
- 🔁 Upsert logic using PostgreSQL `ON CONFLICT`
- 💻 Command-line interface using `argparse`
- 🧩 Modular API, transformation, database, and SQL components
- 🔐 Configuration-based API and database credentials

---

# 🏗️ Architecture

```text
                         USER
                          │
                          ▼
                Command Line Arguments
                  ┌───────┼────────┐
                  │       │        │
                 Term  Location   Price
                  │       │        │
                  └───────┼────────┘
                          ▼
                   BusinessSearch
                          │
                          ▼
                 Yelp Fusion API
                          │
                          ▼
                    JSON Response
                          │
                          ▼
                 Parse & Transform
                          │
                          ▼
                  DatabaseDriver
                          │
                          ▼
                     PostgreSQL
                          │
                          ▼
                    yelp.business
```

---

# 🔄 ETL Workflow

The project follows:

```text
EXTRACT → TRANSFORM → LOAD
```

## 1️⃣ Extract — Yelp Fusion API

Business data is extracted from the Yelp Fusion API using:

```text
term
location
price
```

Example:

```bash
python driver.py --term food --location Montreal --price 4
```

The API returns structured JSON containing business information such as:

- Business ID
- Business name
- Categories
- Rating
- Review count
- Coordinates
- Address
- Price
- Phone number
- Yelp URL
- Image URL

---

## 2️⃣ Transform — JSON Data Processing

The Yelp API response contains nested structures such as:

```text
coordinates
├── latitude
└── longitude

location
└── display_address

categories
├── category 1
├── category 2
└── category 3
```

The `BusinessSearch` class converts this nested response into a flat, database-ready record.

Transformations include:

- Extracting latitude and longitude
- Combining category titles
- Combining address components
- Extracting rating and review count
- Extracting price and phone information
- Preparing values for PostgreSQL
- Escaping single quotes in business names

```text
Yelp JSON
    │
    ▼
BusinessSearch
    │
    ├── Extract fields
    ├── Flatten nested data
    ├── Format categories
    └── Format location
    │
    ▼
Structured Business Record
```

---

## 3️⃣ Load — PostgreSQL

The transformed records are loaded into PostgreSQL.

The project creates:

```text
Schema:
yelp

Table:
yelp.business
```

```text
Transformed Data
      │
      ▼
DatabaseDriver
      │
      ▼
PostgreSQL
      │
      ▼
yelp.business
```

---

# 🌐 Yelp Fusion API

The Yelp Fusion API is the external data source.

The project uses the business search endpoint and sends parameters such as:

```text
term
location
price
```

Example:

```text
term     = restaurants
location = Montreal
price    = 2
```

This makes the pipeline reusable for different business searches.

---

# 🐍 Python Application Flow

The main entry point is:

```text
driver.py
```

Execution flow:

```text
driver.py
    │
    ▼
Parse CLI Arguments
    │
    ▼
BusinessSearch
    │
    ▼
Yelp API Request
    │
    ▼
Retrieve Business Results
    │
    ▼
Parse & Transform Results
    │
    ▼
DatabaseDriver
    │
    ├── Create Schema
    ├── Create Table
    └── Insert / Update Records
    │
    ▼
PostgreSQL
```

---

# 🔎 BusinessSearch

File:

```text
businesssearch.py
```

The `BusinessSearch` class handles business search results returned by the Yelp API.

### Responsibilities

- Build API search parameters
- Request business data
- Process API results
- Extract required fields
- Flatten nested JSON
- Return structured business records

Example:

```python
BusinessSearch(
    term="food",
    location="Montreal",
    price=4
)
```

---

# 🌐 Request Module

File:

```text
request.py
```

The request module handles the HTTP communication with the Yelp API.

Separating API communication from business-data processing keeps the application modular:

```text
HTTP Request Logic
        │
        ▼
Request Module
        │
        ▼
BusinessSearch
```

---

# 🔐 API Authentication

File:

```text
auth.py
```

Authentication configuration is used when communicating with Yelp.

The API key is supplied through an authorization header.

```text
API Key
   │
   ▼
Authorization Header
   │
   ▼
Yelp API
```

Real API credentials should never be committed to a public repository.

---

# 🗃️ PostgreSQL Database

PostgreSQL is the destination database.

The project creates:

```text
yelp
  │
  └── business
```

The `business_id` is the primary key.

---

# 🧱 Database Schema

The `yelp.business` table contains:

| Column | Description |
|---|---|
| `business_id` | Unique Yelp business identifier |
| `business_name` | Name of the business |
| `image_url` | Business image URL |
| `url` | Yelp business page URL |
| `review_count` | Number of reviews |
| `categories` | Business categories |
| `rating` | Yelp rating |
| `latitude` | Geographic latitude |
| `longitude` | Geographic longitude |
| `price` | Price level |
| `location` | Business address |
| `phone` | Business phone number |

---

# 🔁 Upsert / Conflict Handling

The project uses PostgreSQL:

```sql
ON CONFLICT (business_id)
DO UPDATE
```

This prevents duplicate business records when the same business is retrieved multiple times.

```text
             Business ID
                  │
                  ▼
         Does record exist?
            /          \
          YES           NO
           │             │
           ▼             ▼
        UPDATE         INSERT
```

This allows existing business information to be refreshed when new API data is received.

---

# 🗄️ DatabaseDriver

File:

```text
databasedriver.py
```

The `DatabaseDriver` class manages PostgreSQL operations.

### Responsibilities

- Read database configuration
- Establish PostgreSQL connection
- Create database cursor
- Execute SQL queries
- Create Yelp schema
- Create business table

The project uses:

```python
psycopg2
```

for PostgreSQL connectivity.

---

# 🧠 SQL Query Management

File:

```text
queries.py
```

SQL statements are centralized in this module.

It contains queries for:

```text
CREATE SCHEMA
CREATE TABLE
INSERT BUSINESS
UPDATE BUSINESS
```

This separates database logic from the application workflow.

```text
Application Logic
       │
       ▼
DatabaseDriver
       │
       ▼
queries.py
       │
       ▼
PostgreSQL
```

---

# 💻 Command-Line Interface

The application uses Python's `argparse` module.

### Required arguments

```text
--term
--location
```

### Optional argument

```text
--price
```

Price levels:

```text
1 = $
2 = $$
3 = $$$
4 = $$$$
```

Example:

```bash
python driver.py --term food --location Montreal --price 4
```

---

# 📂 Project Structure

```text
Yelp_API_ETL/
│
├── auth.py
├── businesssearch.py
├── databasedriver.py
├── driver.py
├── queries.py
├── request.py
├── config.cfg
└── README.md
```

### `auth.py`

Contains API authentication configuration.

### `businesssearch.py`

Processes Yelp business search results and transforms API data.

### `databasedriver.py`

Manages PostgreSQL connections and query execution.

### `driver.py`

Main application entry point and CLI controller.

### `queries.py`

Contains SQL statements for schema/table creation and business upsert operations.

### `request.py`

Handles HTTP requests to the API.

### `config.cfg`

Stores API and database configuration values.

---

# ⚙️ Configuration

The application uses:

```text
config.cfg
```

Example:

```ini
[KEYS]
CLIENT_KEY=<YOUR CLIENT KEY>
API_KEY=<YOUR API KEY>

[DATABASE]
host=<HOST NAME>
database=<DB NAME>
username=<USER NAME>
password=<PASSWORD>
port=<PORT>
```

### API Configuration

```text
CLIENT_KEY
API_KEY
```

### Database Configuration

```text
host
database
username
password
port
```

> ⚠️ Never commit real API keys, passwords, or other secrets to GitHub.

---

# 🛠️ Setup

## 1. Clone Repository

```bash
git clone https://github.com/VikasKumar281/Yelp_API_ETL.git
cd Yelp_API_ETL
```

## 2. Install Dependencies

If the repository contains `requirements.txt`:

```bash
pip install -r requirements.txt
```

Otherwise, install the core dependencies:

```bash
pip install requests psycopg2-binary
```

## 3. Configure Credentials

Create or update:

```text
config.cfg
```

with your Yelp API and PostgreSQL credentials.

---

# ▶️ How to Run

Example:

```bash
python driver.py --term food --location Montreal --price 4
```

The application will:

```text
1. Read configuration
2. Parse command-line arguments
3. Call Yelp Fusion API
4. Retrieve business results
5. Transform nested JSON
6. Connect to PostgreSQL
7. Create schema if required
8. Create table if required
9. Insert new businesses
10. Update existing businesses
```

---

# 📊 Results

After successful execution, the retrieved business information is stored in:

```text
PostgreSQL
    │
    ▼
yelp.business
```

Stored information includes:

```text
Business Name
Rating
Review Count
Categories
Location
Latitude
Longitude
Price
Phone
Yelp URL
```

### Results Screenshot

The original project README included the following execution-results screenshot:

![Yelp API ETL Results](https://github.com/san089/Udacity-Data-Engineering-Projects/blob/master/Data_Api_to_Postgres/Results.PNG)

The screenshot demonstrates the result/output of the API-to-PostgreSQL workflow.

> **Note:** The image is retained from the original README exactly as provided. If you own a local copy of `Results.PNG`, it is better to add it to this repository under `assets/Results.PNG` and use the local image instead.

---

# 📈 Important Data Engineering Concepts

This project demonstrates:

- **ETL pipeline development**
- **REST API integration**
- **API data extraction**
- **JSON data processing**
- **Data transformation**
- **PostgreSQL data loading**
- **Database schema design**
- **SQL query management**
- **Upsert / conflict handling**
- **Command-line data pipelines**
- **Modular Python architecture**
- **Configuration management**
- **External data ingestion**

---

# 🔒 Security Best Practices

For production environments:

- Never hardcode API keys.
- Never commit database passwords.
- Add sensitive configuration files to `.gitignore`.
- Use environment variables or a secrets manager.
- Use parameterized SQL queries.
- Validate external API responses.
- Add API timeout and retry handling.
- Apply least-privilege PostgreSQL permissions.
- Add structured logging and monitoring.

---

# 🔮 Future Improvements

Potential improvements include:

- Add API pagination
- Add API retry and timeout handling
- Replace SQL string formatting with parameterized queries
- Add automated tests
- Add structured logging
- Add data-quality validation
- Add incremental ingestion
- Add Docker support
- Add Apache Airflow scheduling
- Add CI/CD using GitHub Actions
- Move secrets to environment variables

---

# 📬 Credits

Developed by **Vikas Kumar**

GitHub:

`VikasKumar281`

Project:

`VikasKumar281/Yelp_API_ETL`

---

## ⭐ Project Summary

**Yelp API ETL is a practical Python data engineering pipeline that extracts business data from the Yelp Fusion API, transforms nested JSON responses into structured records, and loads them into PostgreSQL using a modular ETL architecture.**
