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

- Prometheus UI: <http://localhost:9090> 🖥️
- Grafana UI: <http://localhost:3000> (default credentials: admin/admin) 📊
- Alertmanager UI: <http://localhost:9093> ✉️

### 5. Configure Grafana 🛠️

1. Log in to Grafana at <http://localhost:3000>.
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

### How to Use

1. Copy and paste the content above into a **`README.md`** file in your repository.
2. Make sure to include the required configuration files (**`prometheus.yml`**, **`alert.rules.yml`**,**`alertmanager.yml`**) in the appropriate directories.
3. Customize as needed for your specific environment.

This **`README.md`** provides an overview, configuration details, and instructions for setting up the monitoring stack with Docker Compose, with the added touch of emojis to make the documentation more engaging and user-friendly.

To add more alerts to your Prometheus setup, you can define additional alert rules in the **`alert.rules.yml`** file. Here's a step-by-step guide on how to add more alerts:

#### 1. Open the **`alert.rules.yml file`**

This is where your alert rules are defined. Each rule consists of conditions that trigger an alert when those conditions are met.

#### 2. Add a new alert rule

You can define multiple alerts by adding more rules under the **`groups`** section in the **`alert.rules.yml`** file. Here's an example of adding a few different alerts:

#### Example: Memory Usage Alert

```bash
groups:
  - name: example
    rules:
    - alert: HighMemoryUsage
      expr: node_memory_Active_bytes / node_memory_MemTotal_bytes * 100 > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Instance {{ $labels.instance }} high memory usage"
        description: "{{ $labels.instance }} memory usage is above 80% for the last 5 minutes."
```

#### Example: Disk Usage Alert

```bash
    - alert: HighDiskUsage
      expr: node_filesystem_free_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"} * 100 < 20
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} low disk space"
        description: "{{ $labels.instance }} has less than 20% disk space remaining for the last 5 minutes."
```

#### Example: System Load Alert

```bash
    - alert: HighLoad
      expr: node_load1 > 2 * count(node_cpu_seconds_total{mode="system"})
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} high load"
        description: "{{ $labels.instance }} has a load average greater than twice the number of CPU cores for the last 5 minutes."
```

3. Customize Alert Rules

You can customize these alert rules by adjusting the expr (expression) based on the metrics you want to monitor. Here's a breakdown of key elements:

- alert: The name of the alert.
- expr: The PromQL (Prometheus Query Language) expression that defines the condition for the alert.
- for: The duration for which the condition must be true before triggering the alert.
- labels: Additional metadata for the alert, like severity.
- annotations: Provide details about the alert, including summary and description messages.

#### 4. Validate the Configuration

After adding or modifying alert rules, ensure that the configuration is valid:

```bash
docker-compose exec prometheus promtool check rules /etc/prometheus/alert.rules.yml
```

#### 5. Reload Prometheus Configuration

Prometheus will need to reload the configuration to apply the new alert rules. You can do this by:

- Restarting the Prometheus container:

```bash
docker-compose restart prometheus
```

- Using the Prometheus web UI to reload:
  Navigate to **`http://localhost:9090/-/reload`** (assuming Prometheus is running on **`localhost`**).

#### 6. Test Alerts

Once the configuration is reloaded, Prometheus will evaluate the new alert rules. You can simulate or test the alerts by:

- Triggering conditions manually (e.g., using stress tools to overload the system).
- Monitoring the Alerts tab in Prometheus UI to see active alerts.

#### 7. Integrate with Alertmanager

If you haven't already, ensure that these alerts are routed through Alertmanager for email notifications or other alerting mechanisms. The alerts will automatically be sent if configured correctly in the **`prometheus.yml`** file and Alertmanager's configuration.

#### The complete **`alert.rules.yml`**

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

    - alert: HighMemoryUsage
      expr: node_memory_Active_bytes / node_memory_MemTotal_bytes * 100 > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Instance {{ $labels.instance }} high memory usage"
        description: "{{ $labels.instance }} memory usage is above 80% for the last 5 minutes."

    - alert: HighDiskUsage
      expr: node_filesystem_free_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"} * 100 < 20
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} low disk space"
        description: "{{ $labels.instance }} has less than 20% disk space remaining for the last 5 minutes."

    - alert: HighLoad
      expr: node_load1 > 2 * count(node_cpu_seconds_total{mode="system"})
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "Instance {{ $labels.instance }} high load"
        description: "{{ $labels.instance }} has a load average greater than twice the number of CPU cores for the last 5 minutes."
```

This setup will add various alerts for **CPU**, **memory**, **disk**, and system load. You can modify and expand these rules to meet your specific monitoring needs.
