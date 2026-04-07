# monitor-with-docker
## Prepare Host
Install Docker Engine
```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```
Verify
```sh
docker version
docker compose version
```
## Step1 - Prepare Project Directory
Create subdirectories for configurations
```sh
mkdir -p monitoring-stack/prometheus
touch monitoring-stack/docker-compose.yml
touch monitoring-stack/prometheus/prometheus.yml
```
Show Sturcefile
```sh
monitoring-stack/
├── docker-compose.yml
└── prometheus
    └── prometheus.yml
```
## Step2 - Defining the Docker Compose Configuration
Create a docker-compose.yml file in the project root:
```text
cd monitoring-stack/
```
```text
cat <<EOF | tee docker-compose.yml > /dev/null
version: '3.8'
volumes:
  prometheus_data: {}
  grafana_data: {}

networks:
  monitoring:
    driver: bridge

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    ports:
      - "3000:3000"
    networks:
      - monitoring
    restart: unless-stopped
EOF
```
## Step3 - Configuring Prometheus
Create a prometheus.yml file in the prometheus directory:
```text
cat <<EOF | tee prometheus/prometheus.yml > /dev/null
global:
  scrape_interval: 15s
  evaluation_interval: 15s
scrape_configs:
  # System metrics change frequently, scrape more often
  - job_name: 'node-exporter'
    scrape_interval: 10s
    static_configs:
      - targets: ['node-exporter:9100']
EOF
```
## Step 4 - Launching the Monitoring Stack
Start your monitoring stack:
```text
docker compose up -d
```
Verify all containers are running:
```text
docker ps -a
```
## Step 5 - Access the monitoring interface
```text
Prometheus: http://localhost:9090
Grafana: http://localhost:3000 (login with admin/admin)
NodeExporter: http://localhost:3000/metrics
```

## Step 6 - Add Datasource Prometheus
```text
Login Grafana > Connections > Data sources > Add Data Source
```
## Step 7 - Add Dashboard to Grafana
Grafana Dashboard ID
```text
11074
```










