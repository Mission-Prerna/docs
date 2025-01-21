# Student and Spot Assessment Data API

## Overview

This API provides endpoints to retrieve student and spot assessment data within a specified date range. The data can be filtered by various parameters such as date range, limit, offset, and actor type.

## Student Assessment:- 
Student assessment data done by the Teacher, Mentor, and examiner usually aims to evaluate performance or compliance over a defined period.

## Spot Assessment:-
Spot assessments, done by mentors only, are often used to check adherence to standards or procedures in real-time, potentially identifying issues or irregularities that might not be apparent during planned assessments., data

## Endpoints

### 1. Student Assessment Data

**URL:** `{{base_url}}/data/student-assessment-data`  
**Method:** `GET`  
**Description:** Retrieves student assessment data filtered by date ranges (`from_date`/`to_date` or `synced_from`/`synced_to`), actor type, limit, and offset.

**Query Parameters:**
- **Date Range (Required)**:
  - You must provide exactly one date range: either `from_date` and `to_date` **or** `synced_from` and `synced_to`.
  - If provided, the date range cannot exceed 7 days.
  - Dates should be in `YYYY-MM-DD` format.
  
- **Date Range Parameters**:
  - `from_date` (conditional): Start date for filtering data.
  - `to_date` (conditional): End date for filtering data.
  - `synced_from` (conditional): Start date for filtering data based on sync dates.
  - `synced_to` (conditional): End date for filtering data based on sync dates.

- `limit` (optional): Maximum number of records to return. Default is `100000`.
- `offset` (optional): Offset for the data retrieval, used for pagination.
- `actor_type` (optional): Filters data by actor type (`mentor`, `teacher`, `diet mentor`, or `examiner`). Comparison is case-insensitive. If omitted, data for all actor types is retrieved.

**Usage Rules**:
- **Mutually Exclusive Date Ranges**: Provide only one date range at a time. If both date ranges (`from_date`/`to_date` and `synced_from`/`synced_to`) are included, an error will be returned.
- **Required Date Ranges**: Either `from_date` and `to_date` **or** `synced_from` and `synced_to` must be provided. Omitting both ranges will trigger an error.

**Curl Example:**
```bash
curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-01&to_date=2024-08-08&limit=5&actor_type=mentor' \
--header 'authorization: Bearer <<admin-token>>'
```

**Response:**
The response will include the following fields:
- `phone_no`: Mentor's phone number.
- `class`: The class (grade) of the student.
- `user_type`: Type of actor (e.g., mentor).
- `district_name`: Name of the district.
- `block_name`: Name of the block.
- `udise`: School's UDISE code.
- `created_at`: The timestamp when the assessment was created.
- `synced_at`: The timestamp when the data was synced in DB.
- `result`: The result of the assessment (NIPUN, NOT NIPUN, or Unknown).
- `student`: JSON object containing student details, if available:
  - `student_id`: Unique ID of the student.
  - `student_name`: Name of the student.
  - `gender`: Gender of the student.
  - `roll_no`: Roll number of the student.
  - `class`: Class (grade) of the student.
  - `udise`: UDISE code of the student's school.


### 2. Spot Assessment Data

**URL:** `{{base_url}}/data/spot-assessment-data`  
**Method:** `GET`  
**Description:** Retrieves spot assessment data filtered by the specified date range, limit, and offset.

**Query Parameters:**
- `from_date` (required): Start date for the data filter in `YYYY-MM-DD` format.
- `to_date` (required): End date for the data filter in `YYYY-MM-DD` format.
- `limit` (optional): Maximum number of records to return. Default is `100000`.
- `offset` (optional): Offset for the data retrieval, used for pagination.


**Curl Example:**
```bash
curl --location '{{base_url}}/data/spot-assessment-data?limit=50&from_date=2024-04-01&to_date=2024-06-01' \
--header 'authorization: Bearer <<admin-token>>'
```

**Response:**
The response will include the following fields:
- `phone_no`: Mentor's phone number.
- `student_id`: Unique ID of the student (if available).
- `class`: The class (grade) of the student.
- `user_type`: Type of actor (e.g., mentor).
- `district_name`: Name of the district.
- `block_name`: Name of the block.
- `udise`: School's UDISE code.
- `created_at`: The timestamp when the assessment was created.
- `result`: The result of the assessment (NIPUN, NOT NIPUN).


## API Call Process

1. **Set Date Range:**  
   Ensure that the difference between `from_date` and `to_date` does not exceed 7 days. If it does, the API will return a `400` error.

2. **Limit Records:**  
   By default, the API returns up to `100000` records in a single call. If you need fewer records, you can adjust the `limit` parameter.

3. **Offset Management:**  
   The `offset` parameter is used to manage pagination. Initially, set the `offset` to `0`. If the first call returns data, increment the `offset` by the `limit` value and make another call. Repeat this process until the API returns an empty response, indicating that all available data has been retrieved.  
   
   **Example:**
   - First call: `offset=0`

      **Curl Example:**
     
      ```bash
       curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=0&actor_type=teacher' \
       --header 'authorization: Bearer <admin-token>'
     ```
   - Second call: `offset=100000` (assuming `limit=100000`)
     
      **Curl Example:**
     
      ```bash
       curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=100000&actor_type=teacher' \
       --header 'authorization: Bearer <admin-token>'
     ```
   - Third call: `offset=200000` (assuming `limit=100000`)

      **Curl Example:**
     
      ```bash
         curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=20000&actor_type=teacher' \
         --header 'authorization: Bearer <admin-token>'  
     ```
   - **Continue incrementing** the `offset` by the `limit` value until no more data is returned.

   **Important:** If you change the `from_date` or `to_date`, reset the `offset` to `0`. This ensures you retrieve the complete and correct dataset from the new date range without missing or duplicating records.

## Error Handling

The API includes comprehensive error handling to ensure that requests are processed correctly. Below are the common errors you might encounter:

1. **Invalid Date Format (`400 Bad Request`)**  
   - **Condition:** The `from_date` or `to_date` is either missing or not in the correct `YYYY-MM-DD` format.
   - **Error Message:** `"Invalid date format. Please provide dates in 'YYYY-MM-DD' format."`

2. **Date Range Exceeded (`400 Bad Request`)**  
   - **Condition:** The difference between `from_date` and `to_date` exceeds the maximum allowed range of 7 days.
   - **Error Message:** `"Date range exceeded. The maximum allowed range is 7 days."`

3. **Invalid Limit or Offset (`400 Bad Request`)**  
   - **Condition:** The `limit` or `offset` is not a valid number or is out of the allowed range.
   - **Error Message:** `"Invalid limit or offset value. Please provide a valid number."`

4. **Missing Authorization (`401 Unauthorized`)**  
   - **Condition:** The request does not include a valid `Bearer` token in the authorization header.
   - **Error Message:** `"Authorization token is missing or invalid."`

5. **Forbidden Access (`403 Forbidden`)**  
   - **Condition:** The `Bearer` token provided does not have the necessary permissions to access the data.
   - **Error Message:** `"You do not have permission to access this resource."`

6. **Internal Server Error (`500 Internal Server Error`)**  
   - **Condition:** An unexpected error occurs on the server while processing the request.
   - **Error Message:** `"An unexpected error occurred. Please try again later."`

## Authorization

A valid `Bearer` token is required in the authorization header for all API calls. Ensure that the token has the necessary permissions to access the data.

## Environment Variables

- `DEFAULT_DATA_LIMIT`: Default limit for the number of records returned (default: `100000`).
- `MAX_DATE_RANGE_DAYS`: Maximum allowed days between `from_date` and `to_date` (default: `7` days).

--- 

