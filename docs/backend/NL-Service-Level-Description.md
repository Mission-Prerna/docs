# NL-Backend Service Level Documentation

This document describes the services, their purpose, dependencies, and configuration as defined in the provided `docker-compose.yml` file for the NL-App.

---

## **1. tsdb (TimescaleDB)**
### Purpose:
TimescaleDB is used as the primary database to store application data efficiently with support for time-series data.

### Configuration:
- **Image**: `timescale/timescaledb:latest-pg14`
- **Container Name**: `timescale_psql`
- **Ports**: Exposes port `5432` on host port `8015`.
- **Volumes**:
  - `tdb-data`: Stores PostgreSQL database files.
  - `tsdb.postgresql.conf`: Custom PostgreSQL configuration file.
  - `/home/remote/nl-tsdb-backups`: Directory for database backups.
- **Environment Variables**:
  - `POSTGRES_USER`: `user`
  - `POSTGRES_PASSWORD`: `password`
  - `POSTGRES_DB`: `db`
- **Command**: Sets high performance configurations for:
  - `shared_buffers=32GB`
  - `work_mem=32GB`
  - `max_connections=3200`

---

## **2. hasura-tsdb**
### Purpose:
Hasura GraphQL Engine provides an auto-generated GraphQL API for the TimescaleDB.

### Configuration:
- **Image**: `hasura/graphql-engine:v2.23.0`
- **Container Name**: `hasura_tsdb`
- **Ports**: Exposes GraphQL Engine on port `8080` (host port `8011`).
- **Environment Variables**:
  - `HASURA_GRAPHQL_DATABASE_URL`: Connection to TimescaleDB.
  - `HASURA_GRAPHQL_ENABLE_CONSOLE`: `t` (Enables GraphQL Console).
  - `HASURA_GRAPHQL_ADMIN_SECRET`: Authentication secret for GraphQL API.
  - `HASURA_GRAPHQL_PG_CONNECTIONS`: `380` (Max Postgres connections).
  - `HASURA_GRAPHQL_JWT_SECRET`: JWT configuration for authentication.

---

## **3. pgbouncer**
### Purpose:
PgBouncer acts as a connection pooler for the TimescaleDB to handle a high number of client connections.

### Configuration:
- **Image**: `brainsam/pgbouncer:latest`
- **Ports**: Exposes PgBouncer on port `6432`.
- **Volumes**:
  - `userlist.txt`: Contains credentials for PgBouncer.
- **Environment Variables**:
  - `DB_HOST`: `tsdb`
  - `AUTH_TYPE`: `plain`
  - `DEFAULT_POOL_SIZE`: `500`
  - `MAX_CLIENT_CONN`: `50000`
  - `MAX_DB_CONNECTIONS`: `980`
  - `POOL_MODE`: `transaction`

---

## **4. uci-chatbot-helper**
### Purpose:
Provides chatbot-related utility functions and services for the NL-App.

### Configuration:
- **Image**: `samagragovernance/nl-uci-chatbot-helper:3.0.0`
- **Ports**: Exposes on port `3000` (host port `8070`).
- **Environment Variables**: Loaded from `.uci-chatbot-helper.env`.

---

## **5. nginx-nl-apis**
### Purpose:
Acts as a reverse proxy for the NL APIs.

### Configuration:
- **Image**: `nginx:latest`
- **Ports**: Proxies NL APIs traffic from port `4000` to host port `3000`.
- **Volumes**:
  - `nginx-nl-apis.conf`: Nginx configuration file.

---

## **6. kong-nl-apis**
### Purpose:
Manages API gateway functionalities like rate-limiting, logging, and routing.

### Configuration:
- **Image**: `kong:3.5-ubuntu`
- **Ports**:
  - Admin API on `8001` (host `127.0.0.1:8001`).
  - Proxy on `8000` (host `127.0.0.1:8445`).
- **Volumes**:
  - `kong.yml`: Declarative Kong configuration.
- **Environment Variables**:
  - `KONG_DATABASE`: `off` (uses declarative configuration).
  - `KONG_DECLARATIVE_CONFIG`: Path to `kong.yml`.

---

## **7. nl-apis**
### Purpose:
The primary backend service for the NL-App, hosting APIs and business logic.

### Configuration:
- **Image**: `samagragovernance/nl-apis:4.1.3`
- **Dependencies**:
  - `redis-nl-apis`
  - `dragonfly-nl-apis`
  - `tsdb`
- **Environment Variables**: Loaded from `.nl-apis.env`.
- **Scaling**: Allows scaling with workers (`WORKERS` environment variable).

---

## **8. redis-nl-apis**
### Purpose:
Provides in-memory caching and queuing support for the NL APIs.

### Configuration:
- **Image**: `redis:alpine`
- **Ports**: Exposes Redis on `127.0.0.1:8031`.
- **Volumes**:
  - `redis-nl-apis-data`: Persists Redis data.
- **Command**: Configured with:
  - Append-only file for persistence.
  - Replication with read/write permissions.

---

## **9. dragonfly-nl-apis**
### Purpose:
Alternative caching service offering Redis-compatible functionality for high performance.

### Configuration:
- **Image**: `docker.dragonflydb.io/dragonflydb/dragonfly`
- **Ports**: Exposes Dragonfly on `127.0.0.1:8035`.
- **Volumes**:
  - `dragonfly-nl-apis-data`: Persists Dragonfly data.

---

## **10. minio**
### Purpose:
MinIO is used for object storage, providing S3-compatible APIs.

### Configuration:
- **Image**: `bitnami/minio:latest`
- **Ports**:
  - `9000`: S3-compatible API.
  - `9001`: MinIO web interface.
- **Volumes**:
  - `storage`: Persists object data.
- **Environment Variables**:
  - `MINIO_ROOT_USER`: Admin username.
  - `MINIO_ROOT_PASSWORD`: Admin password.

---

## **11. bull-exporter**
### Purpose:
Exports metrics for the Bull queue system to a Prometheus-compatible monitoring system.

### Configuration:
- **Image**: `uphabit/bull_exporter:latest`
- **Ports**: Exposes metrics on port `9674`.
- **Environment Variables**:
  - `EXPORTER_QUEUES`: Monitored queues (e.g., `AssessmentVisitResults`).
  - `EXPORTER_REDIS_URL`: Redis URL for queue monitoring.
  - `EXPORTER_PREFIX`: Prefix for metrics (e.g., `prod`).

---

## **Volumes**
### Persistent Storage:
- **`db-data`**: Stores PostgreSQL data.
- **`tdb-data`**: Stores TimescaleDB data.
- **`logs`**: Stores logs for debugging (commented out in `hasura-tsdb`).
- **`storage`**: Object storage for MinIO.

---

## **Ports Overview**
| Service            | Internal Port | Host Port | Description                    |
|--------------------|---------------|-----------|--------------------------------|
| tsdb              | 5432          | 8015      | TimescaleDB                    |
| hasura-tsdb       | 8080          | 8011      | Hasura GraphQL Engine          |
| pgbouncer         | 6432          | 6432      | Connection Pooler              |
| uci-chatbot-helper| 3000          | 8070      | Chatbot Helper                 |
| nginx-nl-apis     | 4000          | 3000      | Reverse Proxy                  |
| kong-nl-apis      | 8000, 8001    | 8445, 8001| API Gateway                    |
| redis-nl-apis     | 6379          | 8031      | Redis Cache                    |
| dragonfly-nl-apis | 6379          | 8035      | Dragonfly Cache                |
| minio             | 9000, 9001    | 9000, 9001| Object Storage (S3, Web UI)    |
| bull-exporter     | 9538          | 9674      | Bull Queue Metrics Exporter    |

---

## **Dependencies**
- **tsdb**: Core database for most services.
- **redis-nl-apis & dragonfly-nl-apis**: Caching layers for better performance.
- **MinIO**: Object storage for file and data assets.
- **PgBouncer**: Handles high-volume database connections.
- **Hasura**: Provides a GraphQL interface for TimescaleDB.
- **Kong**: Manages API gateway functionalities.

---

This documentation provides a comprehensive overview of the NL-App services, their configurations, dependencies, and purposes.