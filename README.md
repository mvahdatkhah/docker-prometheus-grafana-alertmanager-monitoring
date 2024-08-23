# 🚀 Docker Prometheus Grafana Alertmanager Monitoring

This repository contains a Docker Compose setup for monitoring Linux machines using Prometheus, Grafana, and Node Exporter, with email alerts configured via Alertmanager.
Features ✨

- Prometheus: Scrapes metrics from Node Exporter 📊
- Grafana: Visualizes metrics from Prometheus with customizable dashboards 📈
- Node Exporter: Collects system metrics such as CPU, memory, and disk usage 🖥️
- Alertmanager: Sends alerts based on Prometheus rules via email ✉️

### Prerequisites ⚙️

- Docker 🐳
- Docker Compose

### Getting Started 🛠️

### 1. Clone the Repository 📂

```bash
git clone https://github.com/your_username/docker-prometheus-grafana-alertmanager-monitoring.git
cd docker-prometheus-grafana-alertmanager-monitoring
```

### 2. Configuration 📝

Before starting the services, configure the following files:

#### Prometheus Configuration (prometheus/prometheus.yml)

- Configures Prometheus to scrape metrics from Node Exporter and includes the Alertmanager configuration.

#### Prometheus Alert Rules (prometheus/alert.rules.yml)

- Contains alert rules, such as alerts for high CPU usage.

#### Alertmanager Configuration (alertmanager/alertmanager.yml)

- Configures Alertmanager to send email alerts. Replace the SMTP settings and email addresses with your own.

### 3. Start the Services ▶️

Use Docker Compose to start all the services:

```bash
docker-compose up -d
```

### 4. Access the Services 🌐

- Prometheus UI: http://localhost:9090 🖥️
- Grafana UI: http://localhost:3000 (default credentials: admin/admin) 📊
- Alertmanager UI: http://localhost:9093 ✉️

### 5. Configure Grafana 🛠️

1. Log in to Grafana at http://localhost:3000.
2. Add Prometheus as a data source:

- Go to **Configuration** > **Data Sources** > **Add data source**.
- Select **Prometheus** and set the URL to **`http://prometheus:9090`**.

3. Import a dashboard, such as the **Node Exporter Full** dashboard with ID **`1860`**.

## Configuration Details 🔧

### Docker Compose File (docker-compose.yml)

This file defines the services for Prometheus, Grafana, Node Exporter, and Alertmanager:

```bash
version: '3'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alert.rules.yml:/etc/prometheus/alert.rules.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - "9090:9090"

  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    ports:
      - "9100:9100"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

  alertmanager:
    image: prom/alertmanager:latest
    container_name: alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    ports:
      - "9093:9093"

volumes:
  grafana-data:
```

### Prometheus Configuration (prometheus/prometheus.yml)

```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "/etc/prometheus/alert.rules.yml"
```

### Prometheus Alert Rules (prometheus/alert.rules.yml)

```bash
groups:
  - name: example
    rules:
    - alert: HighCPUUsage
      expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} high CPU usage"
        description: "{{ $labels.instance }} has CPU usage above 80% for the last 5 minutes."
```

### Alertmanager Configuration (alertmanager/alertmanager.yml)

```bash
global:
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'user@example.com'
  smtp_auth_password: 'password'

route:
  receiver: 'email'

receivers:
  - name: 'email'
    email_configs:
      - to: 'your_email@example.com'
```

Replace the SMTP settings and email addresses with your own to enable email alerts.

### Customization 🛠️

You can customize the setup by:

- Adding more services to monitor by modifying the prometheus.yml file.
- Creating additional alert rules in alert.rules.yml.
- Customizing Grafana dashboards according to your needs.

### Stopping the Services ⏹️

To stop the services, run:

```bash
docker-compose down
```

### License 📝

This project is licensed under the MIT License. See the LICENSE file for details.

### How to Use:

1. Copy and paste the content above into a **`README.md`** file in your repository.
2. Make sure to include the required configuration files (**`prometheus.yml`**, **`alert.rules.yml`**,**`alertmanager.yml`**) in the appropriate directories.
3. Customize as needed for your specific environment.

This **`README.md`** provides an overview, configuration details, and instructions for setting up the monitoring stack with Docker Compose, with the added touch of emojis to make the documentation more engaging and user-friendly.
