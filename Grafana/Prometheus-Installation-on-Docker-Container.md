# 🟢 Deploy Prometheus Using Docker and Docker Compose

## 1. 🎯 Objective
This SOP outlines the steps to deploy **Prometheus** using Docker and Docker Compose on a Linux system. It covers creating a dedicated `devops` user, preparing directories, configuring Docker Compose, and running Prometheus as a container.

---

## 2. 📋 Prerequisites
- 🖥️ A Linux server (RHEL, CentOS, Fedora, Ubuntu, or similar)  
- 🔑 Sudo/root access  
- 🌐 Network connectivity to download packages and Docker images  
- 🐳 Docker and Docker Compose installed  
- 🌉 Optional: External Docker network `prometheus_net`

---

## 3. 🔄 Update System Packages
```bash
sudo dnf -y update   # For RHEL/CentOS/Fedora
# OR for Ubuntu/Debian
sudo apt update && sudo -y upgrade
```

---

## 4. 🐳 Install Docker
1. Download Docker installation script:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```
2. Execute the installation script:
```bash
sh get-docker.sh
```
3. Start Docker service and enable on boot:
```bash
sudo systemctl start docker
sudo systemctl enable docker
```
4. Add non-root user to run Docker commands:
```bash
sudo usermod -aG docker devops
```
5. Lock Docker packages to prevent accidental upgrades:
```bash
sudo dnf versionlock add docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo dnf versionlock list
```

---

## 5. 🛠️ Install Docker Compose
1. Download Docker Compose binary:
```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.40.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
2. Set executable permissions:
```bash
sudo chmod 755 /usr/local/bin/docker-compose
```
3. Create symbolic link:
```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
4. Verify installation:
```bash
docker-compose version
```

---

## 6. 👤 Create DevOps User and Assign Docker Permissions
1. Add a new user:
```bash
sudo useradd devops
```
2. Add `devops` user to Docker group:
```bash
sudo usermod -aG docker devops
```
3. Apply group changes:
```bash
newgrp docker
```

---

## 7. 📂 Prepare Prometheus Deployment Directory
```bash
mkdir -p ~/containers/prometheus-server && cd ~/containers/prometheus-server
mkdir prometheus_data prometheus_config prometheus_logs
```

Set proper ownership:
```
docker run --rm -it --entrypoint /bin/sh prom/prometheus:latest
/prometheus $ whoami
nobody
/prometheus $ id nobody
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
```
```bash
sudo chown -R 65534:65534 prometheus_data prometheus_config prometheus_logs
```

Create Docker network:
```bash
docker network create prometheus_net
```

---

## 8. ⚙️ Create Prometheus Configuration File
Create `prometheus.yml` under `prometheus_config/`:
```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
```

---

## 9. 📄 Create Docker Compose File for Prometheus
Create `docker-compose.yml`:
```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus-server
    user: "65534:65534"
    ports:
      - "10.210.2.117:9090:9090"
    restart: always
    volumes:
      - ./prometheus_data:/prometheus:z
      - ./prometheus_config:/etc/prometheus:ro,z
      - ./prometheus_logs:/var/log/prometheus:z
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 10s
      retries: 5
    read_only: true
    deploy:
      resources:
        limits:
          memory: 1024M
          cpus: "0.25"
          pids: 100
    security_opt:
      - no-new-privileges:true
      - label:type:container_t # [For Fedora Based OS Only]
    networks:
      - prometheus_net

networks:
  prometheus_net:
    external: true
```

---

## 10. 🚀 Deploy Prometheus Server
```bash
docker-compose up -d
docker ps
docker-compose ps
```

---

## 11. 📌 Post-Deployment Notes
- 🌐 Prometheus URL: `http://<server_ip>:9090`  
- 📄 Logs: `~/containers/prometheus-server/prometheus_logs/`  
- ⚙️ Config file: `~/containers/prometheus-server/prometheus_config/prometheus.yml`  
- 🗄️ Data path: `~/containers/prometheus-server/prometheus_data/`  

Optional: Check container health:
```bash
docker inspect -f '{{.State.Health.Status}}' prometheus-server
```
