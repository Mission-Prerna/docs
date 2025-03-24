# **NL-App Backend Service Level Documentation**  

This document provides an overview of the **NL-App** backend infrastructure, detailing the services, configurations, dependencies, and interconnections.  

---

## **Table of Contents**  
1. [Service Groups](#service-groups)  
2. [Core NL Services](#1-core-nl-services)  
3. [User Services](#2-user-services)  
4. [Authentication Services](#3-authentication-services)  
5. [Data Collection and Management Services](#4-data-collection-and-management-services)  
6. [Monitoring and Metrics Services](#5-monitoring-and-metrics-services)  
7. [NL Data Lake](#6-nl-data-lake)  
8. [Dependencies](#dependencies)  
9. [Persistent Volumes](#persistent-volumes)  
10. [Ports Overview](#ports-overview)  

---

## **Service Groups**  
1. **Core NL Services**: Backend services for NL-App.  
2. **User Services**: Manages user-related functionalities.  
3. **Authentication Services**: Handles authentication and identity management.  
4. **Data Collection and Management Services**: Facilitates data collection and form management.  
5. **Monitoring and Metrics Services**: Monitors system performance and health.  
6. **NL Data Lake**: Storage and processing infrastructure for large-scale data.  

---

## **1. Core NL Services**  

### **1.1 TimescaleDB (TSDB)**  
- **Purpose**: Time-series database with PostgreSQL compatibility.  
- **Image**: `timescale/timescaledb:latest-pg14`  
- **Ports**: `5432` (exposed as `8015`)  
- **Environment Variables**:  
  - `POSTGRES_USER=user`  
  - `POSTGRES_PASSWORD=password`  
  - `POSTGRES_DB=db`  
- **Volumes**:  
  - `tdb-data`: Stores database files.  
  - `/home/remote/nl-tsdb-backups`: Database backups.  

### **1.2 PgBouncer**  
- **Purpose**: Connection pooling for TimescaleDB.  
- **Image**: `brainsam/pgbouncer:latest`  
- **Ports**: `6432`  
- **Environment Variables**:  
  - `DB_HOST=tsdb`  
  - `DEFAULT_POOL_SIZE=500`  
  - `POOL_MODE=transaction`  
- **Volumes**:  
  - `userlist.txt`: Stores credentials.  

### **1.3 NL API Gateway (Kong)**  
- **Purpose**: API gateway for NL-App.  
- **Image**: `kong:3.5-ubuntu`  
- **Ports**:  
  - `8000` (API Gateway)  
  - `8001` (Admin API)  
- **Environment Variables**:  
  - `KONG_DATABASE=off`  
  - `KONG_DECLARATIVE_CONFIG=/path/to/kong.yml`  
- **Volumes**:  
  - `kong.yml`: API Gateway configuration.  

### **1.4 NL Backend API**  
- **Purpose**: Primary backend service.  
- **Image**: `samagragovernance/nl-apis:4.1.3`  
- **Environment Variables**: Loaded from `.nl-apis.env`.  

### **1.5 Caching (Redis & DragonflyDB)**  
- **Redis**  
  - **Image**: `redis:alpine`  
  - **Ports**: `6379`  
  - **Volumes**: `redis-nl-apis-data`  
- **DragonflyDB**  
  - **Purpose**: High-performance Redis alternative.  
  - **Image**: `docker.dragonflydb.io/dragonflydb/dragonfly`  
  - **Ports**: `6379`  
  - **Volumes**: `dragonfly-nl-apis-data`  

### **1.6 MinIO**  
- **Purpose**: Object storage for large files.  
- **Image**: `bitnami/minio:latest`  
- **Ports**:  
  - `9000` (API)  
  - `9001` (Web UI)  

---

## **2. User Services**  

### **2.1 User Service**  
- **Purpose**: Manages user roles and permissions.  
- **Image**: `samagragovernance/user-service:latest`  
- **Ports**: `8082`  
- **Environment Variables**: Loaded from `.user-service.env`.  

---

## **3. Authentication Services**  

### **3.1 FusionAuth**  
- **Purpose**: Identity and authentication management.  
- **Image**: `fusionauth/fusionauth-app:latest`  
- **Ports**: `9011`  
- **Volumes**:  
  - `fusionauth-config`: Stores configuration.  
- **Environment Variables**:  
  - `DATABASE_URL`: PostgreSQL connection.  

### **3.2 Gatekeeper**  
- **Purpose**: Authentication and authorization service.  
- **Image**: `samagragovernance/gatekeeper-be:latest`  
- **Ports**: `8065:3000`  
- **Volumes**:  
  - `configurations`: Stores authentication settings.  

---

## **4. Data Collection and Management Services**  

### **4.1 ODK Central**  
- **Purpose**: Manages ODK forms and survey data.  
- **Components**:  
  - PostgreSQL (`postgres:9.6`)  
  - Mail Service (`itsissa/namshi-smtp`)  
  - Enketo Web Forms  
  - Redis (`redis:5`)  

### **4.2 ODK-Zip-Nginx**  
- **Purpose**: Handles ODK form uploads in ZIP format.  
- **Image**: `nginx:latest`  
- **Ports**: `${PORT}:80`  
- **Volumes**:  
  - `static`: Directory for serving static files.  
- **Environment Variables**: Loaded from `.env`  

---

## **5. Monitoring and Metrics Services**  

### **5.1 Metrics Stack**  
- **Node Exporter** (`prom/node-exporter:v0.18.1`)  
- **cAdvisor** (`gcr.io/cadvisor/cadvisor:v0.47.0`)  
- **Caddy** (`caddy:2.6.2`)  
- **PGWatch2** (`cybertec/pgwatch2-postgres`)  

---

## **6. NL Data Lake**  

### **6.1 Overview**  
- **Purpose**: Centralized data storage for analytical workloads.  
- **Services**:  
  - Object Storage (MinIO)  
  - Distributed Processing (Spark)  
  - Data Ingestion Pipelines (Kafka, Airflow)  
  - Data Warehouse (ClickHouse, BigQuery)  

---

## **Dependencies**  
- PostgreSQL  
- Redis & DragonflyDB  
- FusionAuth  
- ODK Central  
- Kong API Gateway  
- Metrics Stack  

---

## **Persistent Volumes**  
- `configurations`  
- `secrets`  
- `pgwatch2_pg`  
- `pgwatch2_grafana`  
- `pgwatch2_pw2`  

---

## **Ports Overview**  

| Service            | Internal Port | Host Port | Description                    |  
|--------------------|--------------|-----------|--------------------------------|  
| NL Backend API    | 8081         | 8081      | Primary API service            |  
| NL Kong Gateway   | 8000, 8001   | 8000, 8001| API Gateway & Kong Gateway Admin API        |  
| User Service      | 8082         | 8082      | User management                |  
| FusionAuth        | 9011         | 9011      | Authentication & identity      |  
| Gatekeeper       | 3000         | 8065      | Authentication gateway         |  
| ODK Central      | Various      | Various   | ODK form management            |  
| Metrics (cAdvisor) | 8080        | 8091      | Container metrics monitoring   |  
| Metrics (PGWatch2) | 3000       | 3333      | PostgreSQL performance stats   |  

---

This documentation provides a **structured overview** of the **NL-App backend services**, their configurations, dependencies, and purposes. 🚀
