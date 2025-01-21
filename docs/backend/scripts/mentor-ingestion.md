# Mentor Ingestion Script

This script is for ingesting mentor data from a CSV file into a database. The script handles data validation, creates user records via an external service, and inserts the validated data into the database. It also generates logs for errors encountered during the process.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running the Script](#running-the-script)
  - [Step 1: Load Metadata](#1-load-metadata)
  - [Step 2: Process Mentors CSV](#2-process-mentors-csv)
- [Running in Detached Mode with `nohup`](#running-in-detached-mode-with-nohup)
- [Log Files](#log-files)
- [Error Handling](#error-handling)
- [Environment Variables](#environment-variables)
- [Conclusion](#conclusion)

---

## Prerequisites

Before running the script, ensure you have the following:

- **Python 3.x**: The script is written in Python.
- **Required Python Libraries**: Install using the following command:

  ```bash
  pip install pandas requests python-dotenv
  ```

- **PostgreSQL Database**: Set up and configure the database.
- **External Services**: The script interacts with Hasura for metadata and user management services.

---

## Setup

1. **Clone or Copy the Repository**:
   Clone or copy this repository to your local machine.

2. **Configure the Environment**:
   - Create a `.env` file in the same directory as the script. Include the following variables:
   
     ```bash
     HASURA_SECRET=your_hasura_admin_secret
     HASURA_URL=your_hasura_graphql_endpoint
     CREATE_USER_URL=your_create_user_api_url
     CREATE_USER_AUTHORIZATION=your_create_user_api_token
     APPLICATION_ID=your_application_id
     PASSWORD_VARIABLE=your_default_password
     ```

3. **Prepare the Input File**:
   - Save the mentor data as `mentors.csv` in the same directory as the script. Ensure the file is formatted correctly. You can find the sample CSV file [here](../../../assessets/backend/sample.mentors.csv).

---

## Running the Script

### 1. Load Metadata

The script fetches metadata from Hasura using the `load_metadata` function. Metadata includes districts, blocks, designations, and school details. Ensure the `HASURA_URL` and `HASURA_SECRET` are correctly set in the `.env` file.

### 2. Process Mentors CSV

The script reads `mentors.csv`, validates the data, creates user records, and inserts mentor data into the database. Each record is processed as follows:

- Validate district, block, designation, and school details.
- Create user records in the external service (e.g., FusionAuth).
- Insert data into the `mentor` and `teacher_school_list_mapping` tables.
- Log errors for invalid or failed records.

Run the script using:

```bash
python mentor_ingestion.py
```

---

## Running in Detached Mode with `nohup`

To run the script in the background (detached mode) and ensure it continues running after you close the terminal, use `nohup`:

```bash
nohup python mentor_ingestion.py > mentor_ingestion.log 2>&1 &
```

### Monitor Logs

Check the logs while the script is running:

```bash
tail -f mentor_ingestion.log
```

Stop monitoring the logs using `Ctrl + C`.

---

## Log Files

The script generates log files for errors encountered during the ingestion process. These files are saved in the `logs/` directory and include:

- **FusionAuth Failures**: Logs for failed user creation attempts.
- **Table Failures**: Logs for failed database insertions.
- **Validation Failures**: Logs for records that failed validation.

Each log file is named with the timestamp of the script execution, e.g.:

- `2025-01-14_10_30_45-fa_failed.csv`
- `2025-01-14_10_30_45-table_failed.csv`
- `2025-01-14_10_30_45-validation_failed.csv`

---

## Error Handling

The script handles errors at multiple levels:

1. **Validation Errors**:
   - Missing or invalid district, block, or designation details.
   - Invalid school UDISE codes.

2. **FusionAuth Failures**:
   - Errors during user record creation are logged, and the script continues processing the next record.

3. **Database Errors**:
   - Errors during database insertions are logged, and the script continues processing the next record.

---

## Environment Variables

The script uses the following environment variables from the `.env` file:

- **HASURA_SECRET**: Admin secret for Hasura API.
- **HASURA_URL**: Hasura GraphQL endpoint.
- **CREATE_USER_URL**: URL for creating user records.
- **CREATE_USER_AUTHORIZATION**: Token for authenticating with the user creation API.
- **APPLICATION_ID**: Application ID for user creation.
- **PASSWORD_VARIABLE**: Default password for created users.

---

## Conclusion

This script automates the process of ingesting mentor data from a CSV file, validating the data, creating user records, and inserting it into the database. Logs ensure traceability of errors and allow reprocessing of failed records.

Feel free to modify the script to suit your requirements.
