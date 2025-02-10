# Bulk Examiner Ingestion

This script is designed to ingest examiner data from a CSV file and perform various operations such as creating FusionAuth users, inserting or updating examiners, mapping them to assessment cycles, and invalidating examiners. The script leverages a number of libraries and integrates with external services through API calls.

## Requirements

To run this script, you need to have the following Python packages installed:

- `requests`
- `pandas`
- `python-dotenv`
- `fusionauth-client`

You can install these packages using `pip` by running:

```bash
pip install -r requirements.txt
```

## Configuration

1. **Environment Variables**

   The script uses environment variables for configuration. Create a `.env` file in the `/home/remote/NL-old/hasura/examiner-bulk-ingestion` directory and include the following variables:

     ```plaintext
     FA_API_KEY=<your_fusion_auth_api_key>
     FA_URL=<your_fusion_auth_url>
     HASURA_SECRET=<your_hasura_secret>
     HASURA_URL=<your_hasura_url>
     APPLICATION_ID=<your_application_id>
     PASSWORD_VARIABLE=<your_password_variable>
     ```

    - `FA_API_KEY`: FusionAuth API key for authorizing API requests.
    - `FA_URL`: Base URL for FusionAuth API endpoints.
    - `HASURA_SECRET`: Secret key for authenticating requests to Hasura.
    - `HASURA_URL`: Base URL for the Hasura GraphQL endpoint.
    - `APPLICATION_ID`: FusionAuth application ID for assigning users to the correct application.
    - `PASSWORD_VARIABLE`: Password to use when creating FusionAuth users.

2. **CSV File**

   The script expects a CSV file named `bulk-examiner-ingestion.csv` located at `/home/remote/NL-old/hasura/examiner-bulk-ingestion`. You can find the sample CSV file [here](../../../assessets/backend/bulk-examiner-ingestion-sample.csv).

   The CSV file should contain the following columns:

   - `sr_no`
   - `phone_no`
   - `district_id`
   - `block_id`
   - `designation_id`
   - `actor_id`
   - `officer_name`
   - `subject_of_matter`
   - `area_type`

## Script Overview

The script is located in the `/home/remote/NL-old/hasura/examiner-bulk-ingestion` directory. Below is an overview of its functions:

### `load_metadata()`

Fetches metadata such as districts, blocks, and designations from a Hasura endpoint.

### `get_district_name(district_id)`

Returns the name of a district given its ID.

### `get_designation_name(designation_id)`

Returns the name of a designation given its ID.

### `get_block_name(district_id, block_id)`

Returns the name of a block given its district and block IDs.

### `create_fusion_auth_user(username)`

Creates a user in FusionAuth with the given username.

### `create_examiners(mentor)`

Inserts or updates an examiner record in Hasura.

### `parse_csv(file_path)`

Parses the CSV file, validates data, and creates a list of mentor objects.

### `invalidate_examiners(actor_id)`

Invalidates all active examiners with a specific `actor_id` in both FusionAuth and the Hasura database. This function:
1. **Fetches** all active examiners by `actor_id` from the database.
2. **Deactivates** these users in FusionAuth.
3. **Marks** examiners as inactive in the database only if deactivation in FusionAuth is successful.

### `get_latest_assessment_cycle()`

Fetches the latest assessment cycle ID from Hasura.

### `create_assessment_cycle_district_mentor_mapping(mapping_payload)`

Inserts a mapping of assessment cycles to districts and mentors.

### `log_errors(file_path, data, columns)`

Logs errors to CSV files for further review.

### `main()`

Orchestrates the entire process:
1. Loads metadata.
2. Parses the CSV file.
3. Fetches the latest assessment cycle.
4. Creates FusionAuth users and examiners.
5. Maps examiners to assessment cycles.
6. Invalidates examiners by `actor_id` (if needed).
7. Logs errors.

## Running the Script
Ensure that all necessary environment variables are set and the CSV file is in place inside `/home/remote/NL-old/hasura/examiner-bulk-ingestion` with name `bulk-examiner-ingestion.csv`.

1. **Navigate to Script Directory**

   Change to the `/home/remote/NL-old/hasura/examiner-bulk-ingestion` directory where the script is located:

   ```bash
   cd /home/remote/NL-old/hasura/examiner-bulk-ingestion
   ```

2. **Set Up Virtual Environment**

   It is recommended to use a virtual environment for running the script. Create and activate a virtual environment using the following commands:

   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows use: venv\Scripts\activate
   ```

3. **Install Dependencies**

   With the virtual environment activated, install the required packages:

   ```bash
   pip install -r requirements.txt
   ```

4. **Run the Script in the Background**

   The script may take significant time to execute, so running it in the background using `nohup` is recommended. Execute the following command:

   ```bash
   nohup python3 bulk-examiner-ingestion.py > bulk-examiner-ingestion.log 2>&1 &
   ```

   This command will run the script in the background and output logs to `bulk-examiner-ingestion.log`. To check the script’s progress, you can view this log file.

## Error Handling

Errors during execution are logged into the following CSV files:
- `fa_failed.csv` – Logs failures related to FusionAuth user creation.
- `table_failed.csv` – Logs failures related to examiner creation or updating.
- `assessment_cycle_examiner_mapping_failed.csv` – Logs failures related to mapping examiners to assessment cycles.
- `invalidate_examiners_failed.csv` – Logs failures related to deactivating examiners in FusionAuth or the database.

## License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).
