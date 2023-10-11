# User Data Processing with Apache Airflow

This repository hosts an Apache Airflow DAG (`user_processing`) designed to orchestrate the process of extracting user data from an external API, transforming the data, and then storing it into a Postgres database. This Airflow pipeline demonstrates the power of data automation, ensuring data integrity, and providing a scalable way to handle user data extraction and storage.

## DAG Overview

1. **DAG ID**: `user_processing`
2. **Start Date**: January 1, 2023
3. **Schedule**: Daily
4. **Catch Up**: Disabled

## Tasks in the DAG:

1. **create_table**:
    - **Type**: `SQLExecuteQueryOperator`
    - **Purpose**: This task is responsible for creating a table in a Postgres database to store user data. If the table already exists, this task won't duplicate it.
    - **SQL Query**:
      ```sql
      CREATE TABLE IF NOT EXISTS users(
          firstname TEXT NOT NULL,
          lastname TEXT NOT NULL,
          country TEXT NOT NULL,
          username TEXT NOT NULL,
          password TEXT NOT NULL,
          email TEXT NOT NULL
      )
      ```

2. **is_api_available**:
    - **Type**: `HttpSensor`
    - **Purpose**: This task checks if the external user API is available before proceeding to extract data. It ensures that the subsequent task won't fail due to API unavailability.

3. **extract_user**:
    - **Type**: `SimpleHttpOperator`
    - **Purpose**: This task reaches out to the external API and extracts user data. The response from the API is filtered and logged for verification.
    - **Method**: GET
    - **Response Filtering**: The API's response is parsed to obtain a JSON object.

4. **process_user**:
    - **Type**: `PythonOperator`
    - **Purpose**: After obtaining the user data, this task transforms the data into a structured format suitable for insertion into the database. The processed data is stored temporarily as a CSV file.
    - **Python Callable**: `_process_user`

5. **store_user**:
    - **Type**: `PythonOperator`
    - **Purpose**: This task reads the processed CSV file and inserts the data into the Postgres database using the COPY command for efficient bulk insertion.
    - **Python Callable**: `_store_user`

## Data Flow:
The data flow between tasks is as follows:

`create_table` >> `is_api_available` >> `extract_user` >> `process_user` >> `store_user`

## Purpose of this Airflow Pipeline:
This Airflow pipeline is significant as it automates the workflow of fetching, processing, and storing user data. Such automation ensures that data is consistently updated, transformed, and stored without manual intervention. It's also scalable, allowing for more tasks or modifications in the future if needed.

