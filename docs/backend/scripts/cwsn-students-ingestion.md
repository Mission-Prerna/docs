# CWSN Student Data Ingestion

These scripts to extract data from an Excel file (`cwsn-students.xlsx`), convert the data to JSON format, and then ingest it into a PostgreSQL database. The data extraction script parses the Excel file and creates a JSON file, and the ingestion script loads the data into the database while handling errors.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running the Scripts](#running-the-scripts)
  - [Step 1: Extract Data from Excel](#1-extract-data-from-excel)
  - [Step 2: Ingest Data into PostgreSQL](#2-ingest-data-into-postgresql)
- [Running in Detached Mode with `nohup`](#running-in-detached-mode-with-nohup)
- [Log Files](#log-files)
- [Error Handling](#error-handling)
- [Environment Variables](#environment-variables)
- [Conclusion](#conclusion)

## Prerequisites

Before running the scripts, ensure you have the following installed:

- **Python 3.x**: The scripts are written in Python.
- **Pandas**: Required for reading the Excel file.
- **psycopg2**: PostgreSQL adapter for Python.
- **python-dotenv**: For managing environment variables.
- **PostgreSQL Database**: Set up PostgreSQL to store the data.

You can install the required Python libraries using the following command:

```bash
pip install pandas psycopg2 python-dotenv
```

Ensure your PostgreSQL database is up and running and is properly configured to store the data.

## Setup

1. **Clone or Copy the Repository**:
   Clone or copy this repository to your local machine.

2. **Configure the Environment**:
   - Create a `.env` file in the same directory as the scripts. The `.env` file should contain your PostgreSQL connection string:

   ```bash
   NL_PG_URI=your_postgresql_connection_url
   ```

3. **Excel File**:
   Ensure the Excel file (`cwsn-students.xlsx`) is available in the same directory as the script or adjust the `file_path` variable in `extract-data-from-excel-py.py` to reflect the correct path.

---

## Running the Scripts

### 1. Extract Data from Excel

This script reads the Excel file (`cwsn-students.xlsx`), extracts relevant data (student ID, UDISE code, and activity status), and saves it in JSON format.

To run the script, use the following command:

```bash
python extract-data-from-excel-py.py
```

**Expected Output**: A JSON file (`cwsn-students-data.json`) will be created with the following structure:

```json
[
  {
    "student_id": "12345",
    "udise": 1234567890,
    "is_active": true
  },
  ...
]
```

Make sure the Excel file is named `cwsn-students.xlsx` or update the `file_path` variable in the script if the file name or location is different.

---

### 2. Ingest Data into PostgreSQL

Once the data is extracted into JSON format, use the following script to ingest the data into your PostgreSQL database.

To run the script:

```bash
python ingest-from-extracted-data.py
```

The script will read the `cwsn-students-data.json` file, insert the data into the `cwsn_students` table, and handle any integrity errors (e.g., foreign key violations). Successful and failed records are saved in separate files:

- `cwsn-students-success.json`: Contains records that were successfully inserted/updated.
- `cwsn-students-failure.json`: Contains records that failed to insert or update, along with error messages.

---

## Running in Detached Mode with `nohup`

You can run the scripts in the background (detached mode) using `nohup` so they continue running even if you close the terminal.

### Running the Data Extraction Script in Detached Mode

```bash
nohup python extract-data-from-excel-py.py > extract-data.log 2>&1 &
```

### Running the Data Ingestion Script in Detached Mode

```bash
nohup python ingest-from-extracted-data.py > ingest-data.log 2>&1 &
```

### Check Logs

To monitor the logs of the running process, use:

```bash
tail -f extract-data.log
tail -f ingest-data.log
```

---

## Log Files

Both scripts generate log files for troubleshooting:

- `extract-data.log`: Contains logs from the data extraction process.
- `ingest-data.log`: Contains logs from the data ingestion process.

Logs include detailed information about the success or failure of each record, as well as any errors encountered.

---

## Error Handling

- **Data Extraction**: If the Excel file is not found or if there are issues with reading the file, an error will be logged.
- **Data Ingestion**: If any records fail to insert into the database, they are logged in the `failure` file, and the script continues to process the remaining records.
- **Integrity Errors**: If there is a foreign key violation or other database constraints, the failed record is logged, and the script continues with the next record.

---

## Environment Variables

The `ingest-from-extracted-data.py` script requires the PostgreSQL connection string to be set as an environment variable in a `.env` file.

Example `.env` file:

```bash
NL_PG_URI=postgres://username:password@hostname:port/database
```

---

## Conclusion

This repository automates the process of extracting student data from an Excel file (`cwsn-students.xlsx`), saving it to a JSON file, and then ingesting it into a PostgreSQL database. The scripts handle data extraction, error logging, and PostgreSQL operations efficiently.

Feel free to modify the scripts as per your requirements, such as customizing the database connection or adding more fields to the JSON data.
