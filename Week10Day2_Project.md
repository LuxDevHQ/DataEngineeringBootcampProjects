
### **Week 10, Day 2 - ETL Pipeline & Apache Airflow Scheduling**

#### **📌 Project Overview**
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
