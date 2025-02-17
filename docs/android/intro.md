# Introduction

The **Nipun Lakshya App** is an initiative by the Basic Education Department, Government of Uttar Pradesh, developed under the **NIPUN Bharat** mission. The app enables mentors to conduct **spot assessments** of students in Grades 1 to 3, focusing on foundational education skills such as **reading fluency, comprehension**, and **basic arithmetic problem-solving**. The app is designed to work in both online and offline scenarios, making it highly reliable in remote areas.

### Quick Information
- **Play Store Link**: [Nipun Lakshya App](https://play.google.com/store/apps/details?id=org.samagra.missionPrerna)
- **Current Version**: 2.4.0
- **Languages Used**: Java, Kotlin
- **Architecture**: MVVM, Multi-Module, Offline first
- **Offline-First**: Yes

---

## Technical Highlights

### **MVVM Architecture**
- Ensures a **clean separation of concerns** between UI, business logic, and data layers.
- Improves **scalability**, **testability**, and code **readability**.
- Promotes a more **modular structure**, making the app easier to maintain and extend.

### **Multi-Module Architecture**
- Breaks down the app into **independent modules**, enabling parallel development.
- Reduces **build times** and allows better **code organization**.
- Isolates responsibilities, making debugging simpler and quicker.

### **Offline-First Design**
- Ensures app usability even in areas with **poor connectivity**.
- Preloads all necessary data after login, allowing assessments without active internet.
- Reduces dependency on live servers during assessment tasks.

### **WorkManager for Syncing**
- Ensures **reliable background syncing**, even after app restarts.
- Supports **automatic retries** for failed sync operations.
- Handles tasks efficiently, even in constrained network conditions.

### **Assessment Engine**
- A **server-driven engine** that dynamically handles assessments.
- Flexible design allows for easy configuration of **new assessments** without app updates.
- Designed to work seamlessly in both online and offline scenarios.

### **Chatbot Backed by UCI Tech**
- A **webview-based intelligent chatbot** powered by **UCI technology**.
- Supports advanced features like:
  - Media handling (images, videos, etc.).
  - Notifications for user engagement.
  - Video viewing and **PDF downloading** capabilities.
- Seamlessly integrates into the app for user convenience.

### **Gatekeeper Module**
- Dynamically manages server load at runtime via the front end.
- **Key Features**:
  - Restricts app access for users based on their roles during high server load.
  - Ensures priority roles (e.g., mentors, admins) are given access during peak times.
  - Non-priority roles can be temporarily blocked to reduce server strain.

### **ODK Collect Extension**
- Leverages the **ODK Collect library** for customizable, form-driven assessments.
- Enables server-driven configurations for dynamic workflows.
- Well-tested and proven library for robust data collection.

### **Hilt for Dependency Injection**
- Simplifies dependency injection, reducing boilerplate code.
- Manages the app's dependencies in a **lifecycle-aware** manner.
- Improves **testability** and simplifies the integration of libraries and modules.

---

## Key Modules

### 1. **Login Module**
- **Responsibilities**:
  - Authenticates users and preloads data required for offline usage.
  - Fetches UDISE code, class, and subject-specific data for assessments.
- **Benefits**:
  - Ensures data is ready for offline use immediately after login.
  - Provides a smooth user experience in remote areas.

### 2. **Assessment Module**
- **Responsibilities**:
  - Manages spot assessments for students.
  - Displays scorecards for effective feedback to teachers.
- **Benefits**:
  - Highly flexible, supporting dynamic server-driven assessments.
  - Works reliably in both online and offline environments.

### 3. **Chatbot Module**
- **Responsibilities**:
  - Provides a webview-based chatbot backed by **UCI technology**.
  - Delivers seamless user interaction with features like notifications, media sharing, video viewing, and downloading PDFs.
- **Benefits**:
  - Enhances user engagement through advanced interaction capabilities.
  - Integrates additional features seamlessly into the app's flow.

### 4. **Gatekeeper Module**
- **Responsibilities**:
  - Dynamically restricts app access based on user roles to reduce server strain.
- **Benefits**:
  - Ensures high-priority users can access the app during peak loads.
  - Dynamically manages server-side load without backend reconfigurations.

### 5. **Sync Module**
- **Responsibilities**:
  - Handles all background data synchronization using **WorkManager**.
- **Benefits**:
  - Efficient and reliable syncing, even in poor network conditions.
  - Ensures assessments and data updates are always up-to-date.

### 6. **ODK Integration Module**
- **Responsibilities**:
  - Extends the capabilities of the **ODK Collect library**.
  - Integrates form-based assessments seamlessly.
- **Benefits**:
  - Provides proven, robust support for server-driven data collection.
  - Highly customizable and adaptable to various assessment scenarios.

---

## Libraries and Tools Used

1. **ODK Collect Extension**:
   - Customizable form-driven data collection for dynamic workflows.
   - Supports robust offline and online data collection.

2. **WorkManager**:
   - Handles background tasks with fault tolerance and retry mechanisms.
   - Ensures syncs resume automatically after network issues or app restarts.

3. **Hilt (Dependency Injection)**:
   - Simplifies dependency injection, reducing boilerplate code.
   - Lifecycle-aware dependency management for better resource handling.

4. **UCI Chatbot Integration**:
   - Powers an intelligent chatbot for advanced user interaction.
   - Supports media sharing, notifications, and document handling.

---

## Advantages of the App
- **Scalability**: The multi-module and MVVM architecture make the app easily extendable for future requirements.
- **Reliability**: The offline-first approach ensures seamless functionality in low-connectivity areas.
- **Efficiency**: WorkManager and Hilt improve performance, resource management, and maintainability.
- **Flexibility**: Server-driven Assessment Engine, ODK integration, and Gatekeeper module make the app adaptable to evolving needs.
- **Engagement**: The UCI-powered chatbot enhances user interaction and provides additional utility features.

---

## Conclusion

The **Nipun Lakshya App** is a robust, offline-first solution built using modern architecture and reliable libraries. With features like **ODK Collect integration**, **WorkManager syncing**, a **server-driven assessment engine**, an **intelligent UCI-powered chatbot**, and the **Gatekeeper module**, the app ensures scalability, reliability, and user engagement, making it an essential tool for mentors and educators.
