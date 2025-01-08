# NL API Documentation

## Overview
The NL API provides endpoints for managing assessments, mentor activities, bot telemetry, metadata, and other features. Authentication and role-based access control are implemented to ensure secure usage.

## Authentication
Most endpoints require JWT authentication and are protected with either `JwtAuthGuard` or `JwtAdminGuard`.

## Endpoints

### Health Check
- **Endpoint:** `GET /health`
- **Description**: Checks the health status of Redis and Database services.

---
### Assessment APIs
#### Create Assessment
- **Endpoint:** `POST /api/assessment-visit-results`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Body**: Array of objects with the following structure:
  - `submission_timestamp` (integer, required): Submission timestamp.
  - `grade` (integer, required): Grade (allowed values: 1 to 8).
  - `subject_id` (integer, required): Subject ID (allowed values: SubjectEnum.MATH, SubjectEnum.ENGLISH, SubjectEnum.HINDI).
  - `mentor_id` (integer, required): Mentor ID.
  - `no_of_student` (integer, required): Number of students.
  - `actor_id` (integer, optional): Actor ID.
  - `block_id` (integer, optional): Block ID.
  - `assessment_type_id` (integer, optional): Assessment Type ID.
  - `udise` (integer, required): UDISE code.
  - `app_version_code` (integer, required): App version code.
  - `results` (array, required): Array of student result objects, each containing:
    - `student_name` (string, required): Student name.
    - `competency_id` (integer, required): Competency ID.
    - `subject_id` (integer, optional): Subject ID.
    - `module` (string, required): Module.
    - `end_time` (bigint, required): End time.
    - `is_passed` (boolean, required): Whether the student passed.
    - `start_time` (bigint, required): Start time.
    - `statement` (string, optional): Statement.
    - `achievement` (integer, required): Achievement score.
    - `total_questions` (integer, required): Total questions.
    - `success_criteria` (integer, required): Success criteria.
    - `session_completed` (boolean, required): Whether the session was completed.
    - `is_network_active` (boolean, required): Whether the network was active.
    - `workflow_ref_id` (string, required): Workflow reference ID.
    - `total_time_taken` (integer, optional): Total time taken.
    - `student_session` (UUID, optional): Student session.
    - `student_id` (string, optional): Student ID.
    - `odk_results` (array, required): Array of ODK result objects, each containing:
      - `question` (string, required): Question.
      - `answer` (string, required): Answer.
    - `result_details` (object, optional): Additional result details.
- **Description**: Creates assessment visit results, supports both queue and direct processing.

---
### Mentor APIs
#### Get Visited Schools for Mentor
- **Endpoint:** `GET /api/mentor/schools`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `month` (integer, optional): Month.
  - `year` (integer, optional): Year.
- **Description**: Retrieves school list for mentor visits.

#### Get Mentor Home Screen Metrics
- **Endpoint:** `GET /api/mentor/dashboard-overview`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `month` (integer, optional): Month.
  - `year` (integer, optional): Year.
- **Description**: Get Home screen dashboard metrics for mentors.

#### Get Mentor Details
- **Endpoint:** `GET /api/mentor/details`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `month` (integer, optional): Month.
  - `year` (integer, optional): Year.
- **Description**: Retrieves mentor details.

#### Get Mentor Details V2
- **Endpoint:** `GET /api/v2/mentor/details`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Description**: V2 version of mentor details API.

#### Get Mentor Performance Data
- **Endpoint:** `GET /api/v2/mentor/performance/insights`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Description**: Mentor home screen API.

#### Get Mentor Android and App Action
- **Endpoint:** `GET /api/actions`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `timestamp` (string, required): Timestamp.
- **Description**: Retrieves app and Android app actions for mentors.

#### Get Home Screen Metrics for Teachers
- **Endpoint:** `GET /api/actor/dashboard-overview`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `timestamp` (string, required): Timestamp.
- **Description**: Retrieves home screen metrics for teachers.

#### Update Mentor PIN
- **Endpoint:** `PATCH /api/mentor/pin`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Body**: 
  - `pin` (string, required): Mentor PIN.
- **Description**: Updates mentor PIN.

---
### Bot Telemetry APIs
#### Create Mentor Bot Telemetry
- **Endpoint:** `POST /api/mentor/bot/telemetry`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Body**: Array of objects with the following structure:
  - `botId` (string, required): Bot ID.
  - `action` (integer, required): Action.
- **Description**: Records bot telemetry data.

#### Get Mentor Bot Telemetry
- **Endpoint:** `GET /api/mentor/bot/telemetry`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `action` (integer, required): Action.
- **Description**: Retrieves bot actions for mentors.

#### Get Mentor Bots
- **Endpoint:** `GET /api/mentor/bot`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Description**: Gets all bots for mentors.

---
### Metadata APIs
#### Get Mentor Metadata
- **Endpoint:** `GET /api/metadata`
- **Auth**: Not Required
- **Description**: Retrieves metadata.

#### Get Mentor Metadata v2
- **Endpoint:** `GET /api/v2/metadata`
- **Auth**: Not Required
- **Description**: V2 version of metadata API.

---
### Examiner APIs
#### Get Examiner Performance Insights
- **Endpoint:** `GET /api/examiner/performance/insights`
- **Auth**: Required
- **Roles**: OpenRole, Diet
- **Query**: 
  - `cycle_id` (integer, required): Cycle ID.
- **Description**: Gets performance insights for examiners.

---
### Translation API
- **Endpoint:** `POST /api/bhashini*`
- **Auth**: Not Required
- **Rate Limit**: Configurable throttle limit
- **Body**: 
  - Translation-related fields specific to the Bhashini service.
- **Description**: Bhashini translation service endpoint.

---
### School APIs
#### Upload School List
- **Endpoint:** `POST /api/school/upload`
- **Auth:** Requires Admin Role
- **Description:** Upload a list of schools via an Excel or CSV file.
- **Request Body:** A file containing the school data (Excel/CSV format).
- **Response:**
  ```typescript
  {
    message: string;
    successSchoolList: string[];
    failureSchoolList: string[];
  }
  ```

#### Get School Students
- **Endpoint:** `GET /api/school/:udise/students`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - `udise` (path): School UDISE number.
- **Headers:**
  - `if-none-match`: ETag header for caching.
- **Description:** Get a list of students for the specified school (UDISE).

#### Get School Students Results
- **Endpoint:** `GET /api/school/:udise/students/result`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - `udise` (path): School UDISE number.
  - **Query Parameters:**
    - `grade` (Query): Grade (allowed values: '1', '2', '3').
    - `cycle_id` (Query): The ID of the assessment cycle. (Required if `year` and `month` are not provided).
    - `month` (Query): Month of the assessment cycle (Required if `year` is provided).
    - `year` (Query): Year of the assessment cycle (Required if `month` is provided).
  - **Description**: Get the assessment results for students in a given school.

#### Get School Students Results Summary
- **Endpoint:** `GET /api/school/:udise/students/result/summary`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - `udise` (path): School UDISE number.
  - **Query Parameters:**
    - `grade` (Query): Comma-separated list of grade numbers (e.g., `"1,2,3"`).
  - **Description**: Get a performance summary for students of the specified school.

#### Get School Teacher Performance Insights
- **Endpoint:** `GET /api/school/:udise/teacher/performance/insights`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - `udise` (path): School UDISE number.
- **Description**: Get performance insights for teachers in the specified school.

#### Get School Status
- **Endpoint:** `GET /api/school/status`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - **Query Parameters:**
    - `cycle_id` (Query): The ID of the assessment cycle.
  - **Description**: Get the assessment status of the school.

#### Calculate Examiner Cycle UDISE Result
- **Endpoint:** `POST /api/school/:udise/result/calculate`
- **Auth:** Required
- **Roles:** OpenRole, Diet
- **Parameters:**
  - `udise` (path): School UDISE number.
  - **Query Parameters:**
    - `cycle_id` (Query): The ID of the assessment cycle.
  - **Description**: Calculate the assessment results for a specific school and cycle.

#### Report School Result Fraud
- **Endpoint:** `POST /api/school/:udise/result/report/fraud`
- **Auth:** Required
- **Roles:** OpenRole, Diet (Examiner only)
- **Parameters:**
  - `udise` (path): School UDISE number.
  - `cycle_id` (Query): The ID of the assessment cycle.
- **Description**: Report any fraud related to school assessment results (e.g., if an examiner was forced to submit wrong results).

---
## Admin APIs

### Mentor Management
#### Create Admin
- **Endpoint:** `POST /admin/mentor`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create a new mentor.

- **Body**: An object with the following structure:
  - `phone_no` (string, required): Phone number of the mentor.
  - `officer_name` (string, optional, max length 100): Officer's name.
  - `district_id` (integer, required): District ID.
  - `block_id` (integer, required): Block ID (nullable).
  - `designation_id` (integer, required): Designation ID.
  - `actor_id` (integer, required): Actor type (allowed values: Mentor, Examiner, Teacher, DIET Mentor, Parent).
  - `area_type` (string, optional): Type of area.
  - `is_active` (boolean, optional, default: true): Mentor's active status.
  - `subject_of_matter` (string, optional): Subject of matter.
  - `udise` (integer, optional): School's UDISE number.

#### Create Admin Old
- **Endpoint:** `POST /admin/mentor/old`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create a mentor using legacy format.
- **Body**: An object with the following structure:
  - `phone_no` (string, required): Phone number of the mentor.
  - `officer_name` (string, optional, max length 100): Officer's name.
  - `district_name` (string, required): District name.
  - `block_town_name` (string, optional): Block town name.
  - `designation` (string, required): Designation (allowed values: 'examiner', 'TSPL', 'S.R. G', 'Principal Secretary', etc.).
  - `area_type` (string, optional): Type of area.
  - `subject_of_matter` (string, optional): Subject of matter.
  - `udise` (integer, optional): School's UDISE number.
  - `is_active` (boolean, optional, default: true): Mentor's active status.

#### Get Mentor Details via Phone
- **Endpoint:** `GET /admin/mentor/phone/:phone`
- **Auth:** Required
- **Roles:** Admin
- **Parameters:**
  - `phone` (path): Phone number of the mentor.
- **Description:** Get mentor details by phone number.

#### Clear Mentor Cache
- **Endpoint:** `GET /admin/mentor/clear-cache`
- **Auth:** Required
- **Roles:** Admin
- **Body**: A `MentorClearCacheDto` object with the following structure:
  - `phoneNumbers` (array of strings, optional): List of phone numbers to clear from the cache.

---
### School Management
#### School Geo-fencing
- **Endpoint:** `POST /admin/school/geo-fencing`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Configure school geofencing blacklist.

- **Body**: A `SchoolGeofencingBlacklistDto` object with the following structure:
  - `blacklist` (array of integers, required): List of blacklisted school IDs.
  - `whitelist` (array of integers, required): List of whitelisted school IDs.

---
### Assessment Management
#### Get Assessment Visit Results
- **Endpoint:** `GET /admin/assessment-visit-results`
- **Auth:** Required
- **Roles:** Admin
- **Query Parameters:**
  - `limit` (integer, required): Maximum number of results (allowed range: 50 to 5000).
  - `id` (integer, required): Assessment visit ID.
- **Description:** Retrieve assessment visit results.

---

### Queue Management
#### Pause Assessment Queues
- **Endpoint:** `POST /admin/queues/pause`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Pause all assessment queues.

#### Resume Assessment Queues
- **Endpoint:** `POST /admin/queues/resume`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Resume all assessment queues.

#### Get Count of Queues
- **Endpoint:** `GET /admin/queues/count`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Get count of items in all queues.

#### Get count of failed Queues
- **Endpoint:** `GET /admin/queues/failed-count`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Get count of failed items in all queues.

---
### Student Management
#### Create Students
- **Endpoint:** `POST /admin/students`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create multiple students (max 500).
- **Body**: Array of objects with the following structure:
  - `name` (string, required): Student's name.
  - `gender` (string, required): Gender (allowed values: 'male', 'female').
  - `dob` (date, optional): Date of birth.
  - `admission_date` (date, optional): Admission date.
  - `roll_no` (integer, required): Roll number.
  - `unique_id` (string, required): Unique student ID.
  - `grade` (integer, required): Grade (allowed values: 1 to 8).
  - `father_name` (string, optional): Father's name.
  - `mother_name` (string, optional): Mother's name.
  - `section` (string, optional): Section (allowed values: 'A', 'B', etc.).
  - `udise` (integer, required): School UDISE number.

#### Update Students
- **Endpoint:** `PATCH /admin/students`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Update multiple students (max 500).
- **Body**: Array of objects with the following structure:
  - `name` (string, optional): Student's name.
  - `gender` (string, optional): Gender.
  - `dob` (date, optional): Date of birth.
  - `admission_date` (date, optional): Admission date.
  - `roll_no` (integer, optional): Roll number.
  - `unique_id` (string, required): Unique student ID.
  - `grade` (integer, optional): Grade.
  - `father_name` (string, optional): Father's name.
  - `mother_name` (string, optional): Mother's name.
  - `section` (string, optional): Section.
  - `udise` (integer, optional): School UDISE number.
  - `deleted_at` (date, optional): Date when the student was deleted (nullable).

#### Delete Students
- **Endpoint:** `DELETE /admin/students`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Delete multiple students (max 500).

- **Body**: Array of objects with the following structure:
  - `unique_id` (string, required): Unique student ID to delete.

#### Create Students with Disabilities (CWSN)
- **Endpoint:** `POST /admin/cwsn-students`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create entries for students with disabilities.

- **Body**: Array of objects with the following structure:
  - `student_id` (string, required): Student ID.
  - `udise` (integer, required): School UDISE number.
  - `is_active` (boolean, optional, default: true): Active status.

---

### Assessment Cycle Management
#### Create Assessment Cycle
- **Endpoint:** `POST /admin/assessment-cycle`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create a new assessment cycle.

- **Body**: Array of object with the following structure:
  - `name` (string, required): Name of the assessment cycle.
  - `start_date` (date, required): Start date of the cycle.
  - `end_date` (date, required): End date of the cycle.
  - `class_1_students_count` (integer, required): Number of students in class 1.
  - `class_2_students_count` (integer, required): Number of students in class 2.
  - `class_3_students_count` (integer, required): Number of students in class 3.
  - `nipun_percentage` (integer, required): NIPUN percentage.

#### Create School - Assessment cycle mapping
- **Endpoint:** `POST /admin/assessment-cycle/:cycle_id/district-school-mapping`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Map schools to an assessment cycle.
- **Parameters:**
  - `cycle_id` (path): Assessment cycle ID.
  - `allUdise` (query, optional): Whether to include all schools.
- **Body**: Array of objects with the following structure:
  - `udise` (integer, required): School UDISE number.

#### Create Examiners- District - Assessment cycle mapping
- **Endpoint:** `POST /admin/assessment-cycle/:cycle_id/district-examiner-mapping`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Map examiners to an assessment cycle and district.
- **Parameters:**
  - `cycle_id` (path): Assessment cycle ID.
- **Body**: Array of objects with the following structure:
  - `district_id` (integer, required): District ID.
  - `mentor_id` (integer, required): Mentor ID.

---

### Actor School Blacklist Management
#### Blacklist Actor-School
- **Endpoint:** `POST /admin/school/blacklist`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create school actor blacklist entries.
- **Body**: Array of objects with the following structure:
  - `actor_id` (integer, required): Actor ID.
  - `udise` (integer, required): School UDISE number.
  - `is_active` (boolean, optional, default: true): Whether the blacklist entry is active.

#### Get Blacklisted Schools
- **Endpoint:** `GET /admin/school/blacklist`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Get all school blacklist entries.

#### Updated Blacklisted Schools
- **Endpoint:** `PATCH /admin/school/blacklist`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Update school blacklist entries.
- **Body**: Array of objects with the following structure:
  - `actor_id` (integer, required): Actor ID.
  - `udise` (integer, required): School UDISE number.
  - `is_active` (boolean, optional, default: true): Whether the blacklist entry is active.

#### Delete Blacklisted School
- **Endpoint:** `DELETE /admin/school/blacklist`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Delete school blacklist entries.
- **Body**: Array of objects with the following structure:
  - `actor_id` (integer, required): Actor ID.
  - `udise` (integer, required): School UDISE number.
  - `is_active` (boolean, optional, default: true): Whether the blacklist entry is active.

---

#### Upload Forms Zip file to Minio
- **Endpoint:** `POST /admin/upload-forms-zip`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Upload a zip file containing forms to Minio.
- **Body**: A zip file containing the forms.

---
### Create Competencies
- **Endpoint:** `POST /admin/create-competencies`
- **Auth:** Required
- **Roles:** Admin
- **Description:** Create competencies.
- **Body**: Array of objects with the following structure:
  - `grade` (string, required): Grade.
  - `subject` (string, required): Subject.
  - `learning_outcome` (string, required): Learning outcome.
  - `competency_id` (integer, required): Competency ID.
  - `min_app_version_code` (integer, optional): Minimum app version code.
  - `subject_id` (integer, required): Subject ID.
  - `flow_state` (integer, optional): Flow state.
  - `pass_percent` (integer, optional): Pass percentage.
  - `metadata` (object, optional): Additional metadata.

---

## Notes
- All auth required endpoints require JWT-based authentication.
- Ensure proper authentication and role-based access when calling the APIs.
- For detailed request and response formats, refer to the API schema or examples in the implementation guide.
- Most endpoints use the `MentorInterceptor` for processing.

For additional support, contact the development team.
