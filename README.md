You want to monitor CPU usage of Docker containers.

To do that you need:

**cAdvisor** → exposes container CPU metrics

**Prometheus** → collects those metrics

**Grafana** → shows CPU graphs

**🧱 Architecture (Simple)
**
```
Docker Container
 └── cAdvisor → container CPU metrics
        ↓
    Prometheus → scrape metrics
        ↓
      Grafana → dashboard
```
🧠 Interview Explanation (Perfect Answer)

```
“We use cAdvisor to expose Docker container CPU metrics, Prometheus to scrape those metrics, and Grafana to visualize per-container CPU utilization.”
“We monitor application-specific Docker containers by filtering cAdvisor metrics in Prometheus and visualizing per-container CPU usage in Grafana using PromQL.”
```

**Prerequisites**
```
docker --version
docker compose version
```
**📁 STEP 1: Create Project Folder
**
```
mkdir docker-monitoring
cd docker-monitoring
```
**📄 STEP 2: Create docker-compose.yml**
```
nano docker-compose.yml
```
```
version: "3.8"

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: always

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    restart: always

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8090:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: always
```
Save and exit.

**📄 STEP 3: Create prometheus.yml**
```
nano prometheus.yml
```
```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "cadvisor"
    static_configs:
      - targets: ["cadvisor:8080"]

```
**▶️ STEP 4: Start Monitoring Stack**

```
docker compose up -d
docker ps
```
You MUST see:

prometheus

grafana

cadvisor

**🌐 STEP 5: Open URLs (Browser)**

```
| Tool       | URL                                            |
| ---------- | ---------------------------------------------- |
| Grafana    | [http://localhost:3000](http://localhost:3000) |
| Prometheus | [http://localhost:9090](http://localhost:9090) |
| cAdvisor   | [http://localhost:8080](http://localhost:8090) |
```

**🔐 STEP 6: Login to Grafana**
```
Username: admin

Password: admin

Skip/change password
```
**🔌 STEP 7: Add Prometheus Data Source (IMPORTANT)**

Where: Grafana UI (browser)
Path:

```
Left menu → ⚙️ Settings → Data Sources → Add data source → Prometheus
```
URL field:
```
http://prometheus:9090
```

Click Save & Test

**📊 STEP 8: Monitor Docker Container CPU**

**Option A (Fastest – Recommended)
**
Import ready dashboard

Grafana → Dashboards → Import

Enter dashboard ID:
```
193
```

Select Prometheus

Click Import

🎉 This shows per-container CPU, memory, network

**Option B (Manual PromQL – Interview Useful)
**
Create new panel → PromQL:
```
rate(container_cpu_usage_seconds_total[1m]) * 100
```

👉 Shows CPU usage % per container

Filter a specific container without name:
```
rate(container_cpu_usage_seconds_total{name="grafana"}[1m]) * 100
```
Filter a specific container with name:
```
sum by (name) (
  rate(container_cpu_usage_seconds_total{
    name=~"grafana|prometheus|cadvisor|petmanagement_.*"
  }[1m])
) * 100
```

🛑 Stop Everything

```
docker compose down
```
