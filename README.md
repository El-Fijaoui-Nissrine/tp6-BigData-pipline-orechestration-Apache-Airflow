# Big Data Pipeline with Apache Airflow

##  Project Description

This project is a practical workshop demonstrating the orchestration of a complete Big Data pipeline using Apache Airflow. It simulates the data flow from ingestion to analytics, passing through the Data Lake and Data Lakehouse.

##  Learning Objectives

- Understand an end-to-end Big Data pipeline
- Identify the roles of Data Lake and Data Lakehouse
- Implement an Airflow DAG representing a real pipeline
- Orchestrate different pipeline stages
- Monitor execution via the Airflow interface

##  Pipeline Architecture

The pipeline follows this logic:

```
Sources → Ingestion → Data Lake → Processing → Lakehouse → Analytics
```

### Pipeline Components

1. **Data Lake (RAW zone)**: Storage of raw data
2. **Big Data Processing**: Data cleaning and structuring
3. **Data Lakehouse (CURATED zone)**: Reliable data for analytics
4. **Airflow**: Orchestration of the execution flow

##  Technologies Used

- **Apache Airflow 2.8.1** (via Docker)
- **PostgreSQL 13** (Airflow metadata)
- **Python 3** (processing simulation)
- **Docker & Docker Compose**

## 📁 Project Structure

```
airflow-bigdata-pipeline/
│
├── docker-compose.yml
├── dags/
│   └── bigdata_pipeline.py
└── data/
    ├── raw/              # Data Lake
    ├── processed/        # Intermediate zone
    └── curated/          # Data Lakehouse
```

##  Installation and Configuration

### Prerequisites

- Docker and Docker Compose installed
- Port 8080 available

### Step 1: Create docker-compose.yml file

Create a `docker-compose.yml` file with the following content:

```yaml
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow

  airflow-webserver:
    image: apache/airflow:2.8.1
    depends_on:
      - postgres
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CORE__LOAD_EXAMPLES: "false"
    volumes:
      - ./dags:/opt/airflow/dags
      - ./data:/opt/airflow/data
    ports:
      - "8080:8080"
    command: webserver

  airflow-scheduler:
    image: apache/airflow:2.8.1
    depends_on:
      - airflow-webserver
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    volumes:
      - ./dags:/opt/airflow/dags
      - ./data:/opt/airflow/data
    command: scheduler
```

### Step 2: Initialization (First Time Only)

```bash
# Initialize Airflow database
docker-compose run airflow-webserver airflow db init

# Create admin user
docker-compose run airflow-webserver airflow users create \
  --username airflow \
  --password airflow \
  --firstname Airflow \
  --lastname Admin \
  --role Admin \
  --email admin@airflow.local
```

### Step 3: Start Services

```bash
docker-compose up -d
```

### Step 4: Access Airflow Interface

**URL :**  `http://localhost:8080`

**Credentials:**
- Username: `airflow`
- Password: `airflow`

<img width="1363" height="635" alt="Capture d&#39;écran 2025-12-26 113850" src="https://github.com/user-attachments/assets/d035ff06-d862-40ba-9e63-d873a67af3a9" />


##  DAG Creation

`dags/bigdata_pipeline.py` :

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime
import os

# Path definitions
RAW = "/opt/airflow/data/raw"
PROCESSED = "/opt/airflow/data/processed"
CURATED = "/opt/airflow/data/curated"

# Task 1: Data Ingestion
def ingest_data():
    os.makedirs(RAW, exist_ok=True)
    with open(f"{RAW}/sales.csv", "w") as f:
        f.write("client,amount\nA,100\nB,200\nA,150\nC,300")

# Task 2: Data Validation
def validate_data():
    if not os.path.exists(f"{RAW}/sales.csv"):
        raise ValueError("Missing data")

# Task 3: Data Transformation
def transform_data():
    os.makedirs(PROCESSED, exist_ok=True)
    with open(f"{RAW}/sales.csv") as fin, \
         open(f"{PROCESSED}/sales_clean.csv", "w") as fout:
        fout.write(fin.read())

# Task 4: Load into Data Lakehouse
def load_lakehouse():
    os.makedirs(CURATED, exist_ok=True)
    with open(f"{PROCESSED}/sales_clean.csv") as fin, \
         open(f"{CURATED}/sales_curated.csv", "w") as fout:
        fout.write(fin.read())

# Task 5: Analytics
def analytics():
    print("Data ready for BI / Machine Learning")

# DAG Definition
with DAG(
    dag_id="bigdata_pipeline_complete",
    start_date=datetime(2024, 1, 1),
    schedule_interval="@daily",
    catchup=False,
    description="Complete Big Data pipeline orchestrated with Airflow"
) as dag:
    
    t1 = PythonOperator(task_id="ingest", python_callable=ingest_data)
    t2 = PythonOperator(task_id="validate", python_callable=validate_data)
    t3 = PythonOperator(task_id="transform", python_callable=transform_data)
    t4 = PythonOperator(task_id="load_lakehouse", python_callable=load_lakehouse)
    t5 = PythonOperator(task_id="analytics", python_callable=analytics)
    
    # Dependencies definition
    t1 >> t2 >> t3 >> t4 >> t5
```

##  Pipeline Usage

### 1. DAG Visualization

**list of DAGs:**

<img width="1365" height="230" alt="Capture d&#39;écran 2025-12-26 114426" src="https://github.com/user-attachments/assets/6964e301-bd69-4e3b-82b4-eef395e5de8f" />


### 2. Pipeline Activation

Activate the DAG by clicking the ON/OFF button to the left of the name.



### 3. Graph Visualization

<img width="1004" height="191" alt="Capture d&#39;écran 2025-12-26 114519" src="https://github.com/user-attachments/assets/cf01a59f-a4fd-4830-befb-757035d11d8b" />

### 4. Execution Statistics

<img width="1365" height="506" alt="Capture d&#39;écran 2025-12-26 114901" src="https://github.com/user-attachments/assets/247dae11-2c0e-4df9-8aca-28f46f5a1035" />



- Total runs
- Average execution duration
- Success/failure count
- Run history

##  Results Verification

After successful pipeline execution, verify the creation of files:

```
data/
├── raw/
│   └── sales.csv
├── processed/
│   └── sales_clean.csv
└── curated/
    └── sales_curated.csv
```

<img width="287" height="215" alt="Capture d&#39;écran 2025-12-26 115500" src="https://github.com/user-attachments/assets/432ed583-21c1-4864-8a8c-cb7b4e77ccb2" />




##  The 5 Pipeline Tasks

| Task | Description | Zone |
|------|-------------|------|
| **ingest** | Raw data ingestion | Data Lake (raw) |
| **validate** | Data existence validation | - |
| **transform** | Cleaning and transformation | Processed |
| **load_lakehouse** | Loading curated data | Data Lakehouse (curated) |
| **analytics** | Preparation for BI/ML | - |

##  Stop Services

```bash
docker-compose down
```

##  Restart (Subsequent Uses)

```bash
docker-compose up -d
```

##  Key Points

- **Data Lake**: Stores raw data without transformation
- **Data Lakehouse**: Stores structured and reliable data
- **Orchestration**: Airflow ensures execution order and traceability
- **Scalability**: This pattern can be extended to real Big Data processing (Spark, etc.)
