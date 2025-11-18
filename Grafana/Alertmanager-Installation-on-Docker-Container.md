# 🟢 Deploy Alertmanager Using Docker and Docker Compose

## 1. 🎯 Objective
This SOP outlines the steps to deploy **Alertmanager** using Docker and Docker Compose on a Linux system. It covers preparing directories, configuring Docker Compose, and running Alertmanager as a container.

---

## 2. 📋 Prerequisites
- 🖥️ A Linux server (RHEL, CentOS, Fedora, Ubuntu, or similar)  
- 🔑 Sudo/root access  
- 🌐 Network connectivity to download packages and Docker images  
- 🐳 Docker and Docker Compose installed  
- 🌉 External Docker network `prometheus_net` (shared with Prometheus)

---

## 3. 📂 Prepare Alertmanager Deployment Directory
```bash
mkdir -p ~/containers/alertmanager && cd ~/containers/alertmanager
mkdir alertmanager_data alertmanager_config alertmanager_logs
```

Set proper ownership:
```
docker run --rm -it --entrypoint /bin/sh prom/alertmanager:latest -c "id"
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
```
```bash
sudo chown -R 65534:65534 alertmanager_data alertmanager_config alertmanager_logs
```

Ensure Docker network exists:
```bash
docker network create prometheus_net
```

---

## 4. ⚙️ Create Alertmanager Configuration File
Create `alertmanager.yml` under `alertmanager_config/`:
```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'default'

receivers:
  - name: 'default'
    # Example: Email, Slack, or webhook configs can go here
    # email_configs:
    #   - to: 'admin@example.com'
```

---

## 5. 📄 Create Docker Compose File for Alertmanager
Create `docker-compose.yml`:
```yaml

services:
  alertmanager:
    image: prom/alertmanager:latest
    container_name: dc1-alertmanager
    user: "65534:65534"
    ports:
      - "192.168.20.126:9093:9093"
    restart: always
    environment:
      HTTP_PROXY: http://192.168.20.126:8080
      HTTPS_PROXY: http://192.168.20.126:8081
      NO_PROXY: localhost,127.0.0.1
    volumes:
      - ./alertmanager_data:/alertmanager:z
      - ./alertmanager_config:/etc/alertmanager:ro,z
      - ./alertmanager_logs:/var/log/alertmanager:z
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:9093/-/healthy"]
      interval: 30s
      timeout: 10s
      retries: 5
    read_only: true
    deploy:
      resources:
        limits:
          memory: 512M
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

## 6. 🚀 Deploy Alertmanager
```bash
docker-compose up -d
docker ps
docker-compose ps
```

---

## 7. 📌 Post-Deployment Notes
- 🌐 Alertmanager URL: `http://<server_ip>:9093`  
- 📄 Logs: `~/containers/alertmanager/alertmanager_logs/`  
- ⚙️ Config file: `~/containers/alertmanager/alertmanager_config/alertmanager.yml`  
- 🗄️ Data path: `~/containers/alertmanager/alertmanager_data/`  

Optional: Check container health:
```bash
docker inspect -f '{{.State.Health.Status}}' alertmanager
```
