## Airflow API Integration Documentation

### Overview

This documentation provides a comprehensive guide on the Airflow project structure and the process for integrating new APIs into the existing framework. The project is designed to fetch data from various APIs, process it, and store it in BigQuery. The current implementation includes integration with the Builtwith API, with provisions for future integrations such as Zoominfo.

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

- **builtwith.py**: This file defines the Directed Acyclic Graph (DAG) for fetching and processing data from the Builtwith API. DAGs orchestrate the workflow and ensure that tasks are executed in the correct order with proper dependencies.

#### Core Modules

- **api_integrations**: This directory contains submodules for handling API integrations, data handling, schema definitions, and utility functions. It is the core of the project where all API-related logic is implemented.
  - **data_handlers**: Contains common data handling functions (`common.py`) and helper functions (`helper.py`). These modules provide functions to fetch data from BigQuery, send alerts to Slack, and perform other essential operations.
  - **schema**: Defines the schema for the output tables for each API (`builtwith_schema.py`). This ensures that the data is structured correctly when stored in BigQuery.
  - **service_providers**: Contains the implementation for each API service provider (`builtwith.py` for Builtwith, `zoominfo.py` for future integration). These modules handle the specific logic for interacting with each API, such as authentication, data fetching, and data transformation.
  - **utils**: Configuration settings (`config.py`). This module contains configuration parameters that are used throughout the project, such as API URLs, table names, and other constants.

#### Base Modules

- **create_default_args.py**: Provides default arguments for the DAGs. This module includes configurations like retry policies, start dates, and other DAG settings that are common across different workflows.

### Detailed Component Descriptions

#### data_handlers/helper.py

- **initialize_logging**: Configures the logging settings to output debug level logs, which helps in monitoring the DAG execution and debugging issues.
- **get_variable**: Retrieves the value of an Airflow variable, allowing for dynamic configuration based on the environment or other factors.
- **send_alert_to_slack**: Sends an alert message to a specified Slack channel. This is used for error notifications and monitoring.
- **get_bqhook**: Creates a BigQuery hook instance, which is used to interact with BigQuery for data insertion and retrieval.
- **create_temp_table**: Creates a temporary table in BigQuery. This is useful for staging data before it is moved to the final destination table.
- **load_data_from_dataframe**: Loads data from a pandas DataFrame into BigQuery. This function handles the data transformation and insertion process.
- **load_temp_table_to_main_tables**: Moves data from a temporary table to the main table in BigQuery, ensuring that the data is properly stored in its final destination.

#### data_handlers/common.py

- **fetch_prospect_data**: Fetches prospect data from BigQuery based on a specified SQL query. This function retrieves the data needed for processing from the prospect table.
- **fetch_users_data**: Fetches user data from BigQuery. Similar to `fetch_prospect_data`, but retrieves data from the user table.
- **choose_task_group**: Determines which task group to execute based on the presence of an `interval_days` configuration. This function helps in dynamically altering the DAG execution flow.

#### service_providers/builtwith.py

- **Builtwith Class**: Implements methods specific to the Builtwith API.
  - **get_jwt_token**: Retrieves the JWT token required for API authentication.
  - **create_prospect_payload**: Creates the request payload for fetching prospect data from the API.
  - **fetch_prospect_data_from_api**: Fetches prospect data from the API and processes it.
  - **insert_prospect_data_to_bq**: Inserts the processed prospect data into BigQuery.
  - **create_users_payload**: Creates the request payload for fetching user data from the API.
  - **fetch_users_data_from_api**: Fetches user data from the API and processes it.
  - **insert_users_data_to_bq**: Inserts the processed user data into BigQuery.

#### schema/builtwith_schema.py

- **Builtwith Schema**: Defines the schema for the Builtwith API output tables. This ensures that the data structure is consistent and adheres to the expected format in BigQuery.

#### utils/config.py

- **CONFIG**: Contains configuration settings for different APIs, including URLs, table names, and other constants. This module allows for easy modification and addition of new configurations.

### Adding a New API Integration (e.g., Zoominfo)

To integrate a new API such as Zoominfo, follow these steps:

1. **Add Service Provider Implementation**
   - Create a new file `zoominfo.py` in the `core/api_integrations/service_providers/` directory.
   - Implement the required methods for the Zoominfo API, similar to the `builtwith.py` implementation. These methods will handle tasks such as authentication, payload creation, data fetching, and data insertion.

2. **Define Schema**
   - Create a new schema definition file for Zoominfo in the `core/api_integrations/schema/` directory.
   - Define the schema for the output tables, ensuring that it matches the data structure expected from the Zoominfo API.

3. **Update Configurations**
   - Add the configuration settings for Zoominfo in the `core/api_integrations/utils/config.py` file.
   - Include necessary parameters such as API URL, table IDs, dataset IDs, and other relevant configurations.

4. **Update DAG**
   - Create a new DAG file `zoominfo.py` in the `dags/` directory to define the workflow for the Zoominfo API.
   - Structure the DAG to include tasks for token creation, payload creation, data fetching, and data insertion, following the pattern used in `builtwith.py`.

5. **Update Common Functions (if necessary)**
   - If there are any common functions or utilities that need to be updated to support the new API, modify the `common.py` or `helper.py` files accordingly.

6. **Testing**
   - Ensure all functions are correctly implemented and tested individually.
   - Test the entire DAG to ensure it runs smoothly and processes the data as expected.

### Conclusion

This document provides a comprehensive overview of the existing project structure and the steps required to integrate a new API into the system. By following these guidelines, you can extend the functionality of the Airflow project to include new data sources seamlessly.
