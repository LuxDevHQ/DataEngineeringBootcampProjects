
### **Week 10, Day 2 - ETL Pipeline & Apache Airflow Scheduling**

#### **📌 Project 1 Overview**
This project involves building an **ETL (Extract, Transform, Load) pipeline** to collect foreign exchange rate data from the **Central Bank of Kenya API**, transform it using **Pandas**, store it in a database of your choice, and schedule the workflow using **Apache Airflow**.

By the end of this project, you should be able to:
- Extract data from an API
- Transform and clean the data using Python (Pandas)
- Load the data into **PostgreSQL, MySQL, MongoDB, Redshift, Cassandra, or S3**
- Automate the ETL workflow using **Apache Airflow**

---

### **🛠 Step 1: Extract Data from API**
We will fetch exchange rate data from the Central Bank of Kenya API.

#### **📌 Task**
- Use the `requests` library to send a `POST` request to fetch exchange rate data.
- Convert the response into a Pandas DataFrame.

#### **🔹 Sample Python Code**
```python
import requests
import pandas as pd
from datetime import datetime

# API Endpoint
url = "https://www.centralbank.go.ke/wp-admin/admin-ajax.php?action=get_wdtable&table_id=193"

# Get today's date
today = datetime.now()
date_range = today.strftime('%d/%m/%Y')

# Define form payload
form_data = {
    "draw": "3",
    "columns[0][data]": "0",
    "columns[0][name]": "date_r",
    "columns[0][searchable]": "true",
    "columns[0][orderable]": "true",
    "columns[0][search][value]": date_range,
    "sRangeSeparator": "~"
}

# Make the API request
response = requests.post(url, data=form_data)
data = response.json()

# Convert to Pandas DataFrame
df = pd.DataFrame(data["data"], columns=["Date", "Currency Code", "New Mean"])
print(df.head())  # Display sample data
```

### **🛠 Step 2: Load Data into a Database**
Now that we have the extracted data, store it in a database of your choice.

#### **📌 Task**
Choose a database: PostgreSQL, MySQL, MongoDB, Redshift, Cassandra, or S3.

Write Python code to insert data into the database.

#### **🔹 Example: Insert into PostgreSQL**
```python
from sqlalchemy import create_engine

# Define database connection (update credentials)
engine = create_engine('postgresql://user:password@localhost:5432/exchange_rates')

# Load data into PostgreSQL
df.to_sql('exchange_rates', engine, if_exists='append', index=False)
print("Data successfully inserted into PostgreSQL!")
```

#### **🔹 Example: Save Data to Amazon S3 (CSV Format)**
```python
import boto3

s3 = boto3.client('s3')
csv_buffer = df.to_csv(index=False)

# Upload to S3
s3.put_object(Bucket='your-bucket-name', Key='exchange_rates.csv', Body=csv_buffer)
print("Data successfully uploaded to S3!")
```

### **🛠 Step 3: Automate with Apache Airflow**
Now, automate the ETL process using Apache Airflow.

#### **📌 Task**
Install Apache Airflow if not installed:

```bash
pip install apache-airflow
```

Create a DAG (Directed Acyclic Graph) to schedule the ETL process.

#### **🔹 Example: Airflow DAG**
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import subprocess

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2025, 3, 25),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

def run_etl():
    subprocess.run(["python3", "exchange_rate_etl.py"])

dag = DAG(
    'exchange_rate_pipeline',
    default_args=default_args,
    description='ETL pipeline for exchange rates',
    schedule_interval='@daily'
)

task1 = PythonOperator(
    task_id='run_etl_script',
    python_callable=run_etl,
    dag=dag,
)
```

Save this file as `exchange_rate_dag.py` inside your Airflow DAGs folder.

Start Apache Airflow:

```bash
airflow scheduler
airflow webserver
```

Access the Airflow UI at http://localhost:8080 and enable the DAG.

### **✅ Submission Guidelines**
Each student should:

- Push the full project (code, scripts, and workflows) to a GitHub repository.
- Include a README.md with:
  - Project explanation
  - Steps to run the script and DAG
  - Database setup instructions
- Ensure that the code is well-documented.

### **📅 Deadline**
Submit by the end of Week 10, Day 3.

### **🚀 Bonus Challenge**
- Extend the project to include data quality checks before inserting into the database.
- Implement logging & monitoring to track ETL job execution.


#### **📌 Project 2 Overview**
In this project, you will **build a data pipeline** that automates the process of:
1. **Extracting** data from an API or public dataset.
2. **Transforming** the data into a structured format.
3. **Loading** the cleaned data into **Azure Blob Storage or AWS S3**.
4. **Scheduling** the workflow using **Apache Airflow**.

By the end of this project, you should be able to:
- Extract and process data from APIs
- Store data in **Azure Blob Storage or AWS S3**
- Automate the process using **Apache Airflow**

---

### **🛠 Step 1: Select Data Source**
Choose a **public dataset** or API as your data source. Below are two options:

#### **🔹 Option 1: Public API - OpenWeather API**
- Fetch weather data using OpenWeatherMap API.
- API Endpoint: `https://api.openweathermap.org/data/2.5/weather`
- Example API Call:
  ```bash
  https://api.openweathermap.org/data/2.5/weather?q=Nairobi&appid=YOUR_API_KEY
  ```
  
### **🛠 Step 2: Extract Data using Python**
Write a Python script to fetch data from OpenWeather API.

#### **🔹 Sample Python Script**
```python
import requests
import json
import pandas as pd
from datetime import datetime

# Define API URL and Key
API_KEY = "your_api_key"
CITY = "Nairobi"
URL = f"https://api.openweathermap.org/data/2.5/weather?q={CITY}&appid={API_KEY}"

# Fetch data
response = requests.get(URL)
data = response.json()

# Extract relevant fields
weather_data = {
    "city": data["name"],
    "temperature": data["main"]["temp"],
    "humidity": data["main"]["humidity"],
    "weather": data["weather"][0]["description"],
    "timestamp": datetime.now().strftime('%Y-%m-%d %H:%M:%S')
}

# Convert to DataFrame
df = pd.DataFrame([weather_data])

# Save as CSV
df.to_csv("weather_data.csv", index=False)
print("Weather data saved successfully!")
```

### **🛠 Step 3: Load Data into Cloud Storage**

#### **🔹 Option 1: Load Data to AWS S3**
Use boto3 to store the file in Amazon S3.

```python
import boto3

# AWS Credentials
AWS_ACCESS_KEY = "your-access-key"
AWS_SECRET_KEY = "your-secret-key"
BUCKET_NAME = "your-bucket-name"

# Initialize S3 Client
s3 = boto3.client(
    "s3",
    aws_access_key_id=AWS_ACCESS_KEY,
    aws_secret_access_key=AWS_SECRET_KEY
)

# Upload File
s3.upload_file("weather_data.csv", BUCKET_NAME, "data/weather_data.csv")
print("File uploaded to S3!")
```

#### **🔹 Option 2: Load Data to Azure Blob Storage**
Use azure-storage-blob to store the file in Azure Blob Storage.

```python
from azure.storage.blob import BlobServiceClient

# Azure Credentials
AZURE_CONNECTION_STRING = "your-azure-connection-string"
CONTAINER_NAME = "your-container-name"

# Initialize Blob Client
blob_service_client = BlobServiceClient.from_connection_string(AZURE_CONNECTION_STRING)
blob_client = blob_service_client.get_blob_client(container=CONTAINER_NAME, blob="weather_data.csv")

# Upload File
with open("weather_data.csv", "rb") as data:
    blob_client.upload_blob(data, overwrite=True)
print("File uploaded to Azure Blob Storage!")
```

### **🛠 Step 4: Automate the Pipeline with Apache Airflow**
Use Apache Airflow to schedule and automate the ETL pipeline.

#### **🔹 Install Apache Airflow**
```bash
pip install apache-airflow
```

#### **🔹 Create Airflow DAG**
Save this file as `weather_etl_dag.py` in your Airflow DAGs folder.

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import subprocess

# Define default args
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2025, 3, 25),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Define Python Functions
def run_weather_etl():
    subprocess.run(["python3", "weather_data_etl.py"])

dag = DAG(
    'weather_etl_pipeline',
    default_args=default_args,
    description='ETL pipeline for weather data',
    schedule_interval='@hourly'
)

task1 = PythonOperator(
    task_id='run_weather_etl',
    python_callable=run_weather_etl,
    dag=dag,
)
```

#### **🔹 Start Apache Airflow**
```bash
airflow scheduler
airflow webserver
```

Open Airflow UI at http://localhost:8080.

Enable the `weather_etl_pipeline` DAG.

### **✅ Submission Guidelines**
Each student should:

- Push the full project (Python scripts, DAGs, and configurations) to a GitHub repository.
- Include a README.md with:
  - Project overview
  - Steps to set up AWS S3/Azure Blob Storage
  - Instructions to run the Airflow DAG
- Ensure that all code is well-documented.

### **📅 Deadline**
Submit by the end of Week 10, Day 3


#### **IMPORTANT SCRIPTS:**
```SQL
-- create schema dataengineering for the Exchange Rates Project
create schema dataengineering;

--create the table
CREATE TABLE dataengineering.exchange_rates (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    currency_code TEXT NOT NULL,
    new_mean DECIMAL(10,4) NOT NULL
);

-- Verify the table is created successfully.
select * from dataengineering.exchange_rates;
```

```PYTHON
import pandas as pd
import psycopg2
from sqlalchemy import create_engine
from datetime import datetime
import requests

DB_NAME = "warehouse"
DB_USER = "****"
DB_PASSWORD = "*****"
DB_HOST = "172.178.131.221"
DB_PORT = "****"

engine = create_engine(f"postgresql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}")

# Fetch Data from Central Bank of Kenya
url = "https://www.centralbank.go.ke/wp-admin/admin-ajax.php?action=get_wdtable&table_id=193"

today = datetime.now()
date_range = today.strftime('%d/%m/%Y')

form_data = {
    "draw": "3",
    "columns[0][data]": "0",
    "columns[0][name]": "date_r",
    "columns[0][searchable]": "true",
    "columns[0][orderable]": "true",
    "columns[0][search][value]": date_range,
    "sRangeSeparator": "~"
}

response = requests.post(url, data=form_data)

# Convert response JSON to Pandas DataFrame
data = response.json()
new_df = pd.DataFrame(data["data"], columns=["Date", "Currency Code", "New Mean"])

# Convert Date column to proper format
new_df["Date"] = pd.to_datetime(new_df["Date"], format="%d/%m/%Y")

# Rename columns to match PostgreSQL table
new_df.columns = ["date", "currency_code", "new_mean"]

# Store DataFrame into PostgreSQL
new_df.to_sql("exchange_rates", engine, if_exists="append", index=False, schema="dataengineering")

print("Data inserted successfully!")
```
