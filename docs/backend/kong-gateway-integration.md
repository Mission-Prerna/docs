# Kong Gateway Integration with Docker

This guide provides instructions for integrating Kong Gateway into a service using Docker. Kong Gateway is an open-source API Gateway and Microservices Management Layer.

## Prerequisites

Before you begin, ensure you have the following installed on your system:
- Docker
- Docker Compose

## Part 1: Kong Integration

### Step 1: Clone the Repository

Clone the repository containing your service and Kong Gateway configuration.

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### Step 2: Set Up Environment Variables

Create a `.env` file in the root directory of your project and add the necessary environment variables. Example:

```dotenv
POSTGRES_USER=kong
POSTGRES_DB=kong
POSTGRES_PASSWORD=kong
KONG_DATABASE=postgres
KONG_PG_HOST=kong-database
KONG_PG_PASSWORD=kong
```

### Step 3: Create Docker Compose File

Create a `docker-compose.yml` file in the root directory of your project. Below are the instructions for both DB and DB-less modes.

#### DB Mode

Refer to the [official Kong Gateway documentation](https://docs.konghq.com/gateway/latest/kong-enterprise/kong-enterprise-edition-vs-community-edition/) for more details.

In this we are proceeding with DB-less Mode
#### DB-less Mode

Refer to the [official Kong Gateway DB-less mode documentation](https://docs.konghq.com/gateway/latest/kong-enterprise/kong-enterprise-edition-vs-community-edition/#declarative-configuration-dbless-mode) for more details.

```yaml
version: '3'

services:
  kong-nl-apis:
    image: kong:3.5-ubuntu
    environment:
      KONG_DATABASE: 'off'
      KONG_DECLARATIVE_CONFIG: /etc/kong/kong.yml
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: '0.0.0.0:8001'
      KONG_PROXY_LISTEN: '0.0.0.0:8000'
    volumes:
      - ./monitoring-nl-apis/kong/kong.yml:/etc/kong/kong.yml
    ports:
      - "127.0.0.1:8445:8000"
      - "127.0.0.1:8001:8001"
    restart: always
```

### Step 4: Create Kong Configuration File

1. Create a `config` folder in the root directory of your project.
2. Inside the `config` folder, create a file named `kong.yml`.
3. Add your Kong configuration to the `kong.yml` file.

Example structure:

```plaintext
your-repo/
├── config/
│   └── kong.yml
├── docker-compose.yml
└── .env
```

### Example Configuration in `kong.yml`

Here is an example of how to define a service, a route, and a route with parameters in the path using regex:

```yaml
_format_version: "3.0"
_transform: true

services:
  - name: example-service
    url: http://example-service:3000

routes:
  - name: example-route
    methods:
      - GET
    paths:
      - /example
    strip_path: false
    service: example-service

  - name: example-route-with-params
    methods:
      - GET
    paths:
      - ~/example/(?<id>\d+)
    strip_path: false
    service: example-service
```

### Explanation of Regex Patterns in Paths

- **`~`**: The tilde (`~`) indicates that the path is using a regex pattern. It is required when using regex in the `paths` field to distinguish it from a standard path.
- **`/(?<id>\d+)`**: This regex pattern matches a segment of the path that consists of one or more digits. 
  - `(?<id>\d+)`: This captures a sequence of digits and names it `id`. The angle brackets (`<>`) denote the name of the captured group, which can be used later in plugins or other configurations.
  
In the example above:
- **`/example/(?<id>\d+)`**: Matches paths like `/example/123`, where `123` is captured as `id`.


For more details on creating Kong configuration, refer to the [Kong Declarative Configuration documentation](https://docs.konghq.com/gateway/latest/production/deployment-topologies/db-less-and-declarative-config/).

### Step 5: Start Services

Run the following command to start the services using Docker Compose:

```bash
docker-compose up -d
```

This command will start the necessary services based on the mode (DB or DB-less) you configured.

### Step 6: Verify Installation

To verify that Kong Gateway is running, you can access the following endpoints:

- Kong Admin API: [http://localhost:8001](http://localhost:8001)
- Kong Proxy: [http://localhost:8000](http://localhost:8000)

### Step 7: Configure Your Service

Now you can configure your service to use Kong Gateway. Typically, this involves setting up routes, services, and plugins through the Kong Admin API or using the declarative configuration in `kong.yml`.

## Part 2: Setup Prometheus Plugin for Monitoring Metrics

### Step 1: Add Prometheus Plugin Configuration

Add the Prometheus plugin to your `kong.yml` configuration file to enable metrics collection.

```yaml
_format_version: "3.0"
_transform: true

services:
  - name: example-service
    url: http://example-service:3000    
    plugins:
      - name: prometheus
        config:
          per_consumer: false
          status_code_metrics: true
          latency_metrics: true
          bandwidth_metrics: true
          upstream_health_metrics: true

routes:
  - name: example-route
    methods:
      - GET
    paths:
      - /example
    strip_path: false
    service: example-service

  - name: example-route-with-params
    methods:
      - GET
    paths:
      - ~/example/(?<id>\d+)
    strip_path: false
    service: example-service
```

### Explanation of Prometheus Plugin Configuration

- **`name: prometheus`**: Specifies the use of the Prometheus plugin, which allows Kong to expose metrics that can be scraped by Prometheus for monitoring.
- **`config:`**: Contains the configuration settings for the Prometheus plugin.
  - **`per_consumer: false`**: Disables metrics for individual consumers. Set to `true` to enable detailed metrics per consumer.
  - **`status_code_metrics: true`**: Enables collection of metrics for HTTP status codes, useful for tracking success and error rates.
  - **`latency_metrics: true`**: Enables collection of latency metrics, which measure the time taken to process requests.
  - **`bandwidth_metrics: true`**: Enables collection of bandwidth metrics, which track data throughput.
  - **`upstream_health_metrics: true`**: Enables collection of health metrics for upstream services.

Refer to the [Prometheus plugin documentation](https://docs.konghq.com/hub/kong-inc/prometheus/) for more details.

### Step 2: Create Prometheus Configuration File

1. Create a `prometheus.yml` file in the root directory of your project.
2. Add the following scrape configuration to collect metrics from Kong.

Example `prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'kong'
    static_configs:
      - targets: ['kong:8001']
```

### Step 3: Create Docker Compose File

Add the Prometheus service to your `docker-compose.yml` file to run Prometheus using Docker.

```yaml
version: '3.7'

services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
```

### Step 4: Start Prometheus

Run the following command to start Prometheus along with other services:

```bash
docker-compose up -d
```

### Step 5: Verify Prometheus Metrics

Access the Prometheus dashboard at [http://localhost:9090](http://localhost:9090) and verify that metrics are being collected from Kong.

## Conclusion

You have successfully integrated Kong Gateway into your service using Docker and set up the Prometheus plugin for monitoring metrics. For more advanced configurations and management, refer to the [Kong documentation](https://docs.konghq.com/) and the [Prometheus plugin documentation](https://docs.konghq.com/hub/kong-inc/prometheus/).