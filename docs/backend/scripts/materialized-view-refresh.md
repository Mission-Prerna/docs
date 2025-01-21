# Materialized View Refresh Script

This script automates the process of refreshing multiple materialized views in a PostgreSQL database using Docker Compose. It logs detailed results to a log file and can run as a background process with `nohup` or be scheduled as a cron job for regular execution.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Running the Script](#running-the-script)
- [Log File](#log-file)
- [Cron Job Execution](#cron-job-execution)
- [Running in Detached Mode with `nohup`](#running-in-detached-mode-with-nohup)
- [Script Workflow](#script-workflow)
- [Logging and Debugging](#logging-and-debugging)
- [Dependencies](#dependencies)
- [Example Cron Job Entry](#example-cron-job-entry)
- [Conclusion](#conclusion)

---

## Prerequisites

Before running the script, ensure you have the following installed:

- **Docker** and **Docker Compose**: The script uses `docker-compose` to execute the refresh commands inside a running container.
- **PostgreSQL**: The script assumes you have a PostgreSQL database running inside a Docker container, and the materialized views are set up in the database.
- **Bash**: This script is intended to be run on a Unix-based system that supports Bash.

Ensure your project is correctly set up with Docker and the necessary materialized views.

---

## Setup

1. **Clone or Copy the Script**:  
   Save the script to a file on your system (e.g., `refresh-materialized-views.sh`).

2. **Permissions**:  
   Make sure the script has execute permissions. You can set the execute permission with the following command:
   ```bash
   chmod +x refresh-materialized-views.sh
   ```

3. **Project Directory**:  
   The script assumes the project directory is located at `/home/remote/NL-old/hasura`. If your project is in a different directory, update the `PROJECT_DIR` variable in the script to reflect the correct path.

4. **Docker Compose Service Name**:  
   The script uses `docker-compose exec` to interact with the PostgreSQL database inside a container. Update `{service}`, `{user}`, and `{db}` placeholders in the script with the appropriate values for your setup.

---

## Running the Script

To run the script manually, execute the following command:

```bash
./refresh-materialized-views.sh
```

The script performs the following steps:

1. **Navigate to Project Directory**:  
   It navigates to the project directory defined in the `PROJECT_DIR` variable. If the directory change fails, the script will exit with an error message.

2. **Refresh Materialized Views**:  
   - **`assessment_result_nl_temp_v2`**: First, it attempts to refresh the `assessment_result_nl_temp_v2` materialized view.
   - **`assessment_result_nl_dashboard_v2`**: If the previous refresh is successful, it proceeds to refresh the `assessment_result_nl_dashboard_v2` materialized view.
   - **`examiner_assessment_dashboard`**: Independently, the script will attempt to refresh the `examiner_assessment_dashboard`.
   - **`examiner_assessment_time_taken_dashboard`**: Lastly, it attempts to refresh the `examiner_assessment_time_taken_dashboard` materialized view.

3. **Error Handling**:  
   If any refresh operation fails, the script logs the failure and skips dependent operations.

4. **Log Output**:  
   Logs for each operation are written to `/tmp/db-mvs-refresh.log`. The log includes timestamps for each action and detailed success or failure messages.

---

## Log File

All logs are written to a log file specified in the `LOG_FILE` variable (`/tmp/db-mvs-refresh.log`). You can view the logs at any time by opening the log file or using the following command:

```bash
cat /tmp/db-mvs-refresh.log
```

---

## Cron Job Execution

To schedule this script as a cron job for automatic execution (e.g., daily at midnight), follow these steps:

1. **Open the Crontab file**:
   ```bash
   crontab -e
   ```

2. **Add an entry to run the script every day at midnight** (adjust the path to your script accordingly):
   ```bash
   0 0 * * * /path/to/refresh-materialized-views.sh >> /path/to/your/logfile.log 2>&1
   ```

3. **Save and exit** the crontab editor. The script will now run automatically at the scheduled time.

> **Note:** You can adjust the schedule by changing the cron timing. For example, to run the script every hour, use `0 * * * *` instead of `0 0 * * *`.

---

## Running in Detached Mode with `nohup`

If you wish to run the script manually in detached mode (background process) and keep it running even if the terminal is closed, you can use `nohup` as follows:

1. **Run the script in detached mode** using `nohup`:
   ```bash
   nohup /path/to/refresh-materialized-views.sh > /path/to/your/logfile.log 2>&1 &
   ```

   This command ensures that the script runs in the background, and the output (both standard and error) is logged to `logfile.log`.

2. **Check the logs**: If you want to see the logs of the running script, use the following command:
   ```bash
   tail -f /path/to/your/logfile.log
   ```

3. **Stop the background process**: To stop the background process, you can kill the process using:
   ```bash
   pkill -f refresh-materialized-views.sh
   ```

---

## Script Workflow

1. **Navigate to Project Directory**:  
   The script first changes to the specified directory (`/home/remote/NL-old/hasura`) to ensure it runs in the correct context.

2. **Refresh Materialized Views**:  
   - **`assessment_result_nl_temp_v2`** is refreshed first.
   - If that is successful, **`assessment_result_nl_dashboard_v2`** is refreshed next.
   - **`examiner_assessment_dashboard`** and **`examiner_assessment_time_taken_dashboard`** are refreshed independently.

3. **Logging**:  
   Logs the success or failure of each refresh operation with timestamps to `/tmp/db-mvs-refresh.log`.

---

## Logging and Debugging

For debugging purposes, the script writes all logs to `/tmp/db-mvs-refresh.log`. You can view the logs in real time using:

```bash
tail -f /tmp/db-mvs-refresh.log
```

If you run the script as a cron job or with `nohup`, logs will still be written to the file, ensuring you can monitor the status of the materialized view refresh process.

---

## Dependencies

- **Docker-Compose**: To manage Docker containers for PostgreSQL.
- **PostgreSQL**: The materialized views must exist in the PostgreSQL database.
- **Cron (optional)**: To schedule the script for periodic execution.
- **nohup (optional)**: For running the script in the background.

---

## Example Cron Job Entry

To run the script every day at midnight and log the output:

```bash
0 0 * * * /path/to/refresh-materialized-views.sh >> /path/to/your/logfile.log 2>&1
```

This will ensure the script runs on a daily basis and logs its output to the specified file.

---

## Conclusion

This script automates the refresh of materialized views in a PostgreSQL database running in a Docker container. By utilizing `docker-compose exec`, the script ensures the views are refreshed efficiently while logging the results for easy troubleshooting.

Ensure you adjust the script for your specific Docker service names, PostgreSQL user, and database, and that your project is correctly configured to run in a Docker environment.