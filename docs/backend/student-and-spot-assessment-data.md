# Student and Spot Assessment Data API

## Overview

This API provides endpoints to retrieve student and spot assessment data within a specified date range. The data can be filtered by various parameters such as date range, limit, offset, and actor type.

---

## Student Assessment

Student assessment data done by the Teacher, Mentor, and Examiner usually aims to evaluate performance or compliance over a defined period.

---

## Spot Assessment

Spot assessments, done by mentors only, are often used to check adherence to standards or procedures in real-time, potentially identifying issues or irregularities that might not be apparent during planned assessments.

---

## Endpoints

### 1. Student Assessment Data

**URL:** `{{base_url}}/data/student-assessment-data`  
**Method:** `GET`  
**Description:** Retrieves student assessment data filtered by date ranges (`from_date`/`to_date` or `synced_from`/`synced_to`), actor type, limit, and pagination using either **offset** or **cursor-based (key-based)** method.

---

**Query Parameters:**

- **Date Range (Required)**:
  - You must provide exactly one date range: either `from_date` and `to_date` **or** `synced_from` and `synced_to`.
  - The date range **cannot exceed 31 days**.
  - Dates should be in `YYYY-MM-DD` format.

- **Date Range Parameters**:
  - `from_date` (conditional): Start date for filtering data.
  - `to_date` (conditional): End date for filtering data.
  - `synced_from` (conditional): Start date based on data sync date.
  - `synced_to` (conditional): End date based on data sync date.

- **Pagination Parameters**:
  - `limit` (optional): Maximum number of records to return. Default is `100000`.
  - `offset` (optional): Offset for the data retrieval (used for **offset-based** pagination).
  - `last_created_at` and `last_mentor_id` (optional): Used for **key-based (cursor)** pagination.

- `actor_type` (optional): Filters data by actor type (`mentor`, `teacher`, `diet mentor`, `examiner`). Case-insensitive.

---

**Usage Rules:**
- **Mutually Exclusive Date Ranges**: Provide only one range (either created_at range or synced_at range).
- **Required Date Range**: Omitting both will cause an error.
- **Pagination Choice**: Either use **offset** or **key-based** parameters — not both.

---

**Curl Example:**

```bash
curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-01&to_date=2024-08-31&limit=5&actor_type=mentor' \
--header 'authorization: Bearer <<admin-token>>'
```

---

**Response Fields:**
- `mentor_id`: Mentor's  ID.
- `phone_no`: Mentor's phone number.
- `class`: The student's class (grade).
- `user_type`: Type of actor (e.g., mentor, teacher).
- `district_name`: District name.
- `block_name`: Block name.
- `udise`: UDISE code of the school.
- `created_at`: When the assessment was created.
- `synced_at`: When the assessment was synced.
- `result`: 'NIPUN', 'NOT NIPUN', or 'Unknown'.
- `student`: JSON object (if student info available):
  - `student_id`
  - `student_name`
  - `gender`
  - `roll_no`
  - `class`
  - `udise`

---

### 2. Spot Assessment Data

**URL:** `{{base_url}}/data/spot-assessment-data`  
**Method:** `GET`  
**Description:** Retrieves spot assessment data filtered by date range and pagination.

---

**Query Parameters:**
- `from_date` (required): Start date (`YYYY-MM-DD`).
- `to_date` (required): End date (`YYYY-MM-DD`).
- `limit` (optional): Default `100000`.
- `offset` (optional): For pagination.

---

**Curl Example:**

```bash
curl --location '{{base_url}}/data/spot-assessment-data?limit=50&from_date=2024-04-01&to_date=2024-04-30' \
--header 'authorization: Bearer <<admin-token>>'
```

---

**Response Fields:**
- `phone_no`
- `student_id`
- `class`
- `user_type`
- `district_name`
- `block_name`
- `udise`
- `created_at`
- `result`

---

# 🚀 API Call Process

To efficiently retrieve large amounts of data without missing or duplicating records, follow the correct **pagination method**:

---

## 1. **Set Date Range**
- Always provide a valid date range.
- **Maximum allowed window: 31 days**.
- Exceeding 31 days will result in a `400` error.

---

## 2. **Limit Records**
- Default maximum: **100000** records per call.
- Adjust the `limit` as needed for your application.

---

## 3. **Pagination Methods**

### ➡️ Offset-Based Pagination

Simple pagination using `limit` and `offset`.

**Steps:**
1. Start with `offset=0`.
2. Increment `offset` by `limit` after each call.
3. Stop when the API returns an empty result.

---

**Example Flow:**

- **First Call**: `offset=0`
  ```bash
  curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=0&actor_type=teacher' \
  --header 'authorization: Bearer <admin-token>'
  ```

- **Second Call**: `offset=100000`
  ```bash
  curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=100000&actor_type=teacher' \
  --header 'authorization: Bearer <admin-token>'
  ```

- **Third Call**: `offset=200000`
  ```bash
  curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=100000&offset=200000&actor_type=teacher' \
  --header 'authorization: Bearer <admin-token>'
  ```

- Continue increasing until no data is returned.

> 🔔 **Note:** Reset `offset=0` whenever you change the date window.

---

### ➡️ Key-Based (Cursor) Pagination

More reliable and recommended for large or dynamic datasets.

**Steps:**
1. First call: No `last_created_at` or `last_mentor_id`.
2. From the last record received, capture `created_at` and `mentor_id`.
3. In the next call, pass those as `last_created_at` and `last_mentor_id`.
4. Repeat the process until no more data.

---

**First Call:**
```bash
curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=1000' \
--header 'authorization: Bearer <admin-token>'
```

**Sample Last Record from Response:**
```json
{
  "mentor_id": 5678,
  "created_at": "2024-08-05T10:45:00"
}
```

**Second Call (cursor-based):**
```bash
curl --location '{{base_url}}/data/student-assessment-data?from_date=2024-08-05&to_date=2024-08-06&limit=1000&last_created_at=2024-08-05T10:45:00&last_mentor_id=5678' \
--header 'authorization: Bearer <admin-token>'
```

---

> 🔔 **Note:**  
> - Do not send `offset` when using cursor-based pagination.
> - Always pick the last record's `created_at` and `mentor_id` for the next page.
> - Reset cursor when changing the date window.

---

# 📋 Pagination Summary

| Method        | How to Paginate                             | Recommended For               |
|---------------|---------------------------------------------|--------------------------------|
| Offset-Based  | `limit` + `offset`                          | Small stable datasets          |
| Key-Based     | `limit` + `last_created_at` + `last_mentor_id` | Large, live-updating datasets |

---

# ❗ Error Handling

The API includes robust error handling:

| Error | Cause | Message |
|------|-------|---------|
| `400 Bad Request` | Invalid/missing dates, invalid limit/offset | "Invalid input." |
| `400 Bad Request` | Both cursor and offset used | "Use either offset or cursor-based pagination" |
| `400 Bad Request` | Date window > 31 days | "Date range exceeded. The maximum allowed range is 31 days." |
| `401 Unauthorized` | Missing/invalid token | "Authorization token is missing or invalid." |
| `403 Forbidden` | Token lacks permission | "You do not have permission to access this resource." |
| `500 Internal Server Error` | Unexpected error | "An unexpected error occurred." |

---

# 🔐 Authorization

All API calls require a valid `Bearer` token in the `Authorization` header.

Example:
```bash
Authorization: Bearer <your-token>
```

---

# ✅ Quick Checklist Before Calling API
- Set valid date range (max 31 days).
- Decide whether to use **offset** or **cursor** pagination.
- Fetch token and add to Authorization header.
- Start with `offset=0` or no cursor for the first page.

---
