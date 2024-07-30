## Airflow API Integration Documentation

### Overview

This documentation provides an overview of the Airflow project structure and a detailed guide on how to integrate new APIs into the existing framework. The project is designed to fetch data from various APIs, process it, and store it in BigQuery. The current implementation includes integration with the Builtwith API. Future integrations, such as with Zoominfo, will follow a similar pattern.

### Project Structure

The project is organized as follows:

```
project_root/
│
├── dags/
│   ├── builtwith.py          # DAG definition for Builtwith API
│   ├── __init__.py           # Package initialization
│
├── core/
│   ├── __init__.py           # Package initialization
│   ├── api_integrations/
│   │   ├── __init__.py       # Package initialization
│   │   ├── data_handlers/
│   │   │   ├── __init__.py   # Package initialization
│   │   │   ├── common.py     # Common data handling functions
│   │   │   ├── helper.py     # Helper functions
│   │   ├── schema/
│   │   │   ├── __init__.py   # Package initialization
│   │   │   ├── builtwith_schema.py # Schema definitions for Builtwith API
│   │   ├── service_providers/
│   │   │   ├── __init__.py   # Package initialization
│   │   │   ├── builtwith.py  # Service provider implementation for Builtwith API
│   │   │   ├── zoominfo.py   # Placeholder for Zoominfo API (future)
│   │   ├── utils/
│   │   │   ├── __init__.py   # Package initialization
│   │   │   ├── config.py     # Configuration settings
│
├── base/
│   ├── __init__.py           # Package initialization
│   ├── create_default_args.py # Default arguments for DAGs
│
├── README.md                 # Project documentation
```

### Key Components

#### DAGs

- **builtwith.py**: Defines the Directed Acyclic Graph (DAG) for fetching and processing data from the Builtwith API.

#### Core Modules

- **api_integrations**: Contains submodules for handling API integrations, data handling, schema definitions, and utility functions.
  - **data_handlers**: Includes common data handling functions (`common.py`) and helper functions (`helper.py`).
  - **schema**: Defines the schema for the output tables for each API (`builtwith_schema.py`).
  - **service_providers**: Contains the implementation for each API service provider (`builtwith.py` for Builtwith, `zoominfo.py` for future integration).
  - **utils**: Configuration settings (`config.py`).

#### Base Modules

- **create_default_args.py**: Provides default arguments for the DAGs.

### Detailed Components and Functions

#### data_handlers/helper.py

- **initialize_logging**: Initializes and configures the logger.
  ```python
  def initialize_logging():
      logging.basicConfig(level=logging.DEBUG, format="%(asctime)s [%(levelname)s] %(message)s")
      logger = logging.getLogger(__name__)
      logger.setLevel(logging.DEBUG)
      return logger
  ```

- **get_variable**: Retrieves the value of an Airflow variable.
  ```python
  def get_variable(key):
      return Variable.get(key)
  ```

- **send_alert_to_slack**: Sends an alert message to a Slack channel.
  ```python
  def send_alert_to_slack(error, data=None, batch=None, dag_name=""):
      slack_webhook_url = CONFIG['common']['slack_webhook_url']
      msg = f"{dag_name} DAG encountered an Error: {error}"
      requests.post(slack_webhook_url, json={"text": msg})
  ```

- **get_bqhook**: Creates a BigQuery hook instance.
  ```python
  def get_bqhook():
      return BigQueryHook()
  ```

- **create_temp_table**: Creates a temporary table in BigQuery.
  ```python
  def create_temp_table(table_id, schema):
      client = get_bqhook().get_client()
      table = bigquery.Table(table_id, schema=schema)
      client.create_table(table)
  ```

- **load_data_from_dataframe**: Loads data from a DataFrame to BigQuery.
  ```python
  def load_data_from_dataframe(table_id, dataframe, schema, dag_name=""):
      client = get_bqhook().get_client()
      client.insert_rows_from_dataframe(table_id, dataframe, selected_fields=schema)
  ```

- **load_temp_table_to_main_tables**: Moves data from a temporary table to the main table.
  ```python
  def load_temp_table_to_main_tables(input_table, output_table):
      client = get_bqhook().get_client()
      client.query(f"INSERT INTO {output_table} SELECT * FROM {input_table}")
  ```

#### data_handlers/common.py

- **fetch_prospect_data**: Fetches prospect data from BigQuery.
  ```python
  def fetch_prospect_data(**kwargs):
      table_id = f"{CONFIG['common']['PROJECT_ID']}.{CONFIG['common']['PROSPECTS_DATASET_ID']}"
      sql_query = f"SELECT DISTINCT(domain), partner_user_id AS user_id FROM {table_id}"
      return get_bqhook().get_pandas_df(sql_query)
  ```

- **fetch_users_data**: Fetches user data from BigQuery.
  ```python
  def fetch_users_data(**kwargs):
      table_id = f"{CONFIG['common']['PROJECT_ID']}.{CONFIG['common']['USERS_DATASET_ID']}"
      sql_query = f"SELECT url, user_id FROM {table_id}"
      return get_bqhook().get_pandas_df(sql_query)
  ```

- **choose_task_group**: Chooses the task group based on interval days.
  ```python
  def choose_task_group(**kwargs):
      conf = kwargs['dag_run'].conf
      if conf.get('interval_days'):
          return 'UsersData.query_data_from_bq'
      else:
          return ['ProspectData.query_data_from_bq', 'UsersData.query_data_from_bq']
  ```

#### service_providers/builtwith.py

- **Builtwith Class**: Implements methods for the Builtwith API.
  ```python
  class Builtwith:
      @staticmethod
      def get_jwt_token():
          return get_variable("builtwith-secret")

      @staticmethod
      def create_prospect_payload(**kwargs):
          # Implementation
          pass

      @staticmethod
      def fetch_prospect_data_from_api(**kwargs):
          # Implementation
          pass

      @staticmethod
      def insert_prospect_data_to_bq(**kwargs):
          # Implementation
          pass

      @staticmethod
      def create_users_payload(**kwargs):
          # Implementation
          pass

      @staticmethod
      def fetch_users_data_from_api(**kwargs):
          # Implementation
          pass

      @staticmethod
      def insert_users_data_to_bq(**kwargs):
          # Implementation
          pass
  ```

#### schema/builtwith_schema.py

- **Builtwith Schema**: Defines the schema for Builtwith API output tables.
  ```python
  builtwith_prospect_schema = [
      {"name": "SpendHistory", "type": "RECORD", "mode": "REPEATED", "fields": [{"name": "D", "type": "INTEGER"}, {"name": "S", "type": "INTEGER"}]},
      {"name": "IsDB", "type": "STRING", "mode": "NULLABLE"},
      # Additional fields
  ]

  builtwith_user_schema = [
      {"name": "SpendHistory", "type": "RECORD", "mode": "REPEATED", "fields": [{"name": "D", "type": "INTEGER"}, {"name": "S", "type": "INTEGER"}]},
      {"name": "IsDB", "type": "STRING", "mode": "NULLABLE"},
      # Additional fields
  ]
  ```

#### utils/config.py

- **CONFIG**: Configuration settings for different APIs.
  ```python
  CONFIG = {
      "builtwith": {
          "Builtwith_url": "https://api.builtwith.com/v21/api.json",
          "TEMP_PROSPECTS_TABLE_ID": "builtwith_prospect",
          "PROSPECTS_DATASET_ID": "PROSPECT_DATASET",
          "USERS_DATASET_ID": "USER_DATASET",
          "PROJECT_ID": "PROJECT_ID",
          "TEMP_DATASET_ID": "builtwith_temp",
          "batch_size": 16,
          "TEMP_USERS_TABLE_ID": "users",
          "USER_OUTPUT_TABLE_ID": "USER_TABLE",
          "PROSPECT_OUTPUT_TABLE_ID": "PROSPECT_TABLE",
          "users_outputfields": ["SpendHistory", "IsDB", "Spend", "Paths", "Meta", "Attributes", "FirstIndexed", "LastIndexed", "Lookup", "SalesRevenue"],
          "prospect_outputfields": ["SpendHistory", "IsDB", "Spend", "Paths", "Meta", "Attributes", "FirstIndexed", "LastIndexed", "Lookup", "SalesRevenue"],
      },
      "common": {
          "PROSPECTS_DATASET_ID": "SOURCE_PROSPECT_DATASET",
          "USERS_DATASET_ID": "SOURCE_USER_DATASET",
          "USERS_TABLE_ID": "SOURCE_USERS_TABLE_ID",
          "PROSPECT_TABLE_ID": "SOURCE_PROSPECT_TABLE_ID

",
          "PROJECT_ID": "PROJECT_ID",
          "GCP_SA": "pipelines-dags@nc-business-intelligence.iam.gserviceaccount.com",
          "slack_webhook_url": "https://hooks.slack.com/services/..."
      }
  }
  ```

### Adding a New API Integration (e.g., Zoominfo)

To integrate a new API such as Zoominfo, follow these steps:

1. **Add Service Provider Implementation**

   Create a new file `zoominfo.py` in the `core/api_integrations/service_providers/` directory. Implement the required methods for the Zoominfo API, similar to the `builtwith.py` implementation.

   ```python
   # core/api_integrations/service_providers/zoominfo.py

   from core.api_integrations.data_handlers.helper import *
   from core.api_integrations.utils.config import CONFIG
   import requests
   import pandas as pd
   from datetime import datetime

   class Zoominfo:
       @staticmethod
       def get_jwt_token():
           pass

       @staticmethod
       def create_payload(**kwargs):
           pass

       @staticmethod
       def fetch_data_from_api(**kwargs):
           pass

       @staticmethod
       def insert_data_to_bq(**kwargs):
           pass
   ```

2. **Define Schema**

   Create a new schema definition file for Zoominfo in the `core/api_integrations/schema/` directory.

   ```python
   # core/api_integrations/schema/zoominfo_schema.py

   zoominfo_schema = [
       {"name": "Field1", "type": "STRING", "mode": "NULLABLE"},
       # Add other fields as required
   ]
   ```

3. **Update Configurations**

   Add the configuration settings for Zoominfo in the `core/api_integrations/utils/config.py` file.

   ```python
   CONFIG["zoominfo"] = {
       "Zoominfo_url": "https://api.zoominfo.com/v1/",
       "TEMP_TABLE_ID": "zoominfo_temp",
       "DATASET_ID": "zoominfo_dataset",
       "TABLE_ID": "zoominfo_table",
       "batch_size": 100,
       "output_fields": ["Field1", "Field2"] # Add other fields as required
   }
   ```

4. **Update DAG**

   Create a new DAG file `zoominfo.py` in the `dags/` directory to define the workflow for the Zoominfo API.

   ```python
   # dags/zoominfo.py

   from airflow import DAG
   from core.api_integrations.service_providers.zoominfo import Zoominfo
   from core.api_integrations.data_handlers.helper import initialize_logging
   from datetime import datetime, timedelta
   from base.create_default_args import create_default_args
   from airflow.operators.dummy_operator import DummyOperator
   from airflow.operators.python_operator import PythonOperator

   logger = initialize_logging()

   default_args = create_default_args(
       datetime(2024, 5, 3),
       depends_on_past=False,
       retries=3,
       retry_delay=timedelta(seconds=5),
   )

   with DAG(
       "zoominfo_dag",
       default_args=default_args,
       description="Fetch and process data from Zoominfo API",
       schedule_interval="0 2 * * *",
       start_date=datetime(2024, 7, 22),
       catchup=False,
   ) as dag:
       start = DummyOperator(task_id="start")
       end = DummyOperator(task_id="end")

       create_jwt_token = PythonOperator(
           task_id="create_jwt_token",
           python_callable=Zoominfo.get_jwt_token
       )

       create_payload = PythonOperator(
           task_id="create_payload",
           python_callable=Zoominfo.create_payload
       )

       fetch_data = PythonOperator(
           task_id="fetch_data_from_api",
           python_callable=Zoominfo.fetch_data_from_api
       )

       insert_data = PythonOperator(
           task_id="insert_data_to_bq",
           python_callable=Zoominfo.insert_data_to_bq
       )

       start >> create_jwt_token >> create_payload >> fetch_data >> insert_data >> end
   ```

5. **Update Common Functions (if necessary)**

   If there are any common functions or utilities that need to be updated to support the new API, modify the `common.py` or `helper.py` files accordingly.

6. **Testing**

   - Ensure all functions are correctly implemented and tested individually.
   - Test the entire DAG to ensure it runs smoothly and processes the data as expected.

### Conclusion

This document provides a comprehensive overview of the existing project structure and the steps required to integrate a new API into the system. By following these guidelines, you can extend the functionality of the Airflow project to include new data sources seamlessly.
