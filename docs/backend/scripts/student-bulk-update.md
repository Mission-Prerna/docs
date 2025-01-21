# Students Bulk Update Script

This Node.js script performs bulk updates for student records in a PostgreSQL database. It reads data from JSON files, deletes existing student records (if applicable), and updates or inserts new records into the database. The script uses **Node.js**, **PostgreSQL**, and **dotenv** for environment configuration.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Configuration](#configuration)
- [Running the Script](#running-the-script)
- [Running the Script in Detached Mode](#running-the-script-in-detached-mode-with-log-file)
- [Process Flow](#process-flow)
- [Error Handling](#error-handling)
- [Folder Structure](#folder-structure)
- [Dependencies](#dependencies)
- [Notes](#notes)
- [Conclusion](#conclusion)

---

## Prerequisites

Before running the script, ensure you have the following installed:

- **Node.js** (version 14 or higher)
- **npm** (Node Package Manager)
- **PostgreSQL** (running and accessible from the server)
- **Access to the PostgreSQL database URI** (configured in `.env` file)

---

## Setup

1. **Install dependencies**:
   Run the following command to install required npm dependencies:
   ```bash
   npm install
   ```

2. **Configure your environment**:
   Create a `.env` file in the root of the project with the following content:
   ```bash
   NL_PG_URI=your_postgresql_database_uri
   CHUNKS=10000  # Number of students to process per batch
   ```
   - `NL_PG_URI`: The connection string for your PostgreSQL database.
   - `CHUNKS`: The batch size for bulk updates and deletions. Default is `10000`.

3. **Ensure the input folder exists**:
   The script expects the student data files to be located in the folder `PrernaStudentData_json`. If this folder doesn't exist, create it and place the student data files (in JSON format) inside it.

---

## Configuration

The configuration for PostgreSQL connection is handled using the `.env` file. Make sure the following configurations are set:

- `NL_PG_URI`: The URI connection string for the PostgreSQL database where the students' data is to be updated.
  
  Example:
  ```bash
  NL_PG_URI=postgres://username:password@host:port/database
  ```

- `CHUNKS`: The batch size for the bulk operations. This determines how many records will be processed per database transaction. 

---

## Running the Script

To run the script, use the following command:

```bash
npm start
```

This will trigger the script, which will:
1. Connect to the PostgreSQL database.
2. Delete existing student records in the database that have a grade greater than 3 (if not already deleted).
3. Read the student data from JSON files.
4. Update or insert students into the database in batches.
5. Write the response of each batch to an output folder `PrernaStudentData_json_response_prod`.

---

## Running the Script in Detached Mode (With Log File)

If you want to run the script in the background and detach it from the terminal session while also saving all logs into a specific file, you can use the `nohup` command with output redirection:

1. In your terminal, navigate to the project directory.
2. Run the script in detached mode using `nohup` and redirect both `stdout` and `stderr` to a log file. Since the script is named `students-update-script.js`, we will name the log file accordingly:

   ```bash
   nohup npm start > students-update-script.log 2>&1 &
   ```

   This command does the following:
   - `nohup`: Runs the script in the background and prevents it from terminating if the terminal is closed.
   - `npm start`: Runs the script via the `start` script defined in `package.json`.
   - `> students-update-script.log`: Redirects `stdout` (standard output) to the `students-update-script.log` file.
   - `2>&1`: Redirects `stderr` (error output) to `stdout`, meaning both regular and error logs will go into the same file.
   - `&`: Runs the process in the background.

3. **View Logs**: To view the logs of the running process, use:

   ```bash
   tail -f students-update-script.log
   ```

   This will display the latest entries in the `students-update-script.log` file as the script runs.

4. **Check Logs After Execution**: Once the script finishes running, you can open the `students-update-script.log` file to inspect the complete logs.

---

## Process Flow

1. **Database Connection**: The script connects to the PostgreSQL database using the URI provided in the `.env` file.

2. **Delete Existing Students**: It first marks all students with a grade greater than 3 as deleted (soft delete).

3. **Read Input Data**: It then reads student data from JSON files located in the `PrernaStudentData_json` folder.

4. **Update Students**: The script updates or inserts student records in the PostgreSQL database in batches (specified by `CHUNKS`).

5. **Write Output**: After each batch of student updates, the script writes the results (success and failure details) to an output file located in `PrernaStudentData_json_response_prod`.

---

## Error Handling

- If any error occurs while connecting to the database, the script will terminate immediately and log the error message.
- Errors during the bulk update process will be logged and the affected students will be marked as failed, with the error message provided.
- The script ensures that database transactions are rolled back in case of failure during any batch update operation.

---

## Folder Structure

The project folder structure is as follows:

```bash
students-bulk-update/
│
├── PrernaStudentData_json/                   # Folder containing input student data files (JSON format)
│
├── PrernaStudentData_json_response_prod/     # Folder for saving the output responses
│
├── students-update-script.js                 # Main script file
│
├── package.json                              # Project metadata and dependencies
│
├── .env                                      # Environment configuration file
│
└── node_modules/                             # npm dependencies
```

---

## Dependencies

This script requires the following npm dependencies:

- **dotenv**: Loads environment variables from the `.env` file.
- **pg**: PostgreSQL client for Node.js used to connect to and interact with the PostgreSQL database.
- **mysql**: (Not used in the script, but included in the package.json).
- **xlsx**: (Not used in the script, but included in the package.json for potential future use).

To install the dependencies, run:

```bash
npm install
```

---

## Notes

- The script assumes that student data is in a JSON format and each record contains the following fields:
  - `unique_id`: Unique identifier for the student.
  - `name`: Name of the student.
  - `gender`: Gender of the student.
  - `roll_no`: Roll number of the student.
  - `grade`: Grade of the student.
  - `udise`: UDISE code of the student.
  
- If a student's `unique_id` exists in the database, the script will update the existing record. If it doesn't exist, the script will insert a new record.

- The output response for each update batch will be saved in the folder `PrernaStudentData_json_response_prod` in the following naming convention:
  - `input_filename-update-response-greater_3.json`

---

## Conclusion

This script automates the process of updating and managing student data in a PostgreSQL database. It handles bulk updates efficiently by processing data in batches, ensuring minimal database load and reducing the risk of timeouts or errors. Ensure proper database connection settings and input data for smooth execution of the script.
