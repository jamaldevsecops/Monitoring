# 🟢 Deploy Node Exporter Using Docker and Docker Compose

## 1. 🎯 Objective
This SOP describes how to deploy **Node Exporter** as a Docker container using Docker Compose.  
Node Exporter exposes system metrics (CPU, memory, disk, network, etc.) that can be scraped by Prometheus for monitoring.

---

## 2. 📋 Prerequisites
- 🐧 Linux server (RHEL, CentOS, Ubuntu, etc.)  
- 🔑 Sudo/root access  
- 🐳 Docker and Docker Compose already installed  
- 👤 `devops` user added to Docker group  
- 🌉 Optional: existing Docker network `prometheus_net`

---

## 3. 📂 Prepare Node Exporter Deployment Directory
Create directories for configuration:
```bash
mkdir -p ~/containers/node-exporter && cd ~/containers/node-exporter
```

If not already created, you may create a shared Docker network:

```bash
docker network create prometheus_net
```

---

## 4. ⚙️ Create Docker Compose File for Node Exporter
Create `docker-compose.yml` under `~/containers/node-exporter/`:

```yaml
services:
  node_exporter:
    image: prom/node-exporter:v1.10.2
    container_name: node-exporter
    user: "65534:65534"
    restart: always
    ports:
      - "10.210.2.117:9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
    read_only: true
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:9100/ || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.15"
          pids: 50
    security_opt:
      - no-new-privileges:true
      - label:type:container_t # [For Fedora Based OS Only]
    networks:
      - prometheus_net

networks:
  prometheus_net:
    external: true
```
Or, 
```yaml
services:
  node_exporter:
    image: prom/node-exporter:v1.10.2
    container_name: node-exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host        # Binds directly to host network
    pid: host                 # Access host processes
    restart: always
    volumes:
      - '/:/host:ro,rslave'  # Full root filesystem, read-only
    read_only: true
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:9100/ || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "0.15"
          pids: 500  # Optional; container sees all host PIDs
```

---

## 5. 🚀 Deploy Node Exporter
Run the following commands:

```bash
docker-compose up -d
docker ps
docker-compose ps
```

---

## 6. ✅ Post-Deployment Verification
1. Verify container status:
   ```bash
   docker inspect -f '{{.State.Health.Status}}' node-exporter
   ```

2. Access Node Exporter metrics endpoint:
   ```
   http://<server_ip>:9100/metrics
   ```

3. Add the Node Exporter target to your Prometheus configuration:
   ```yaml
   - job_name: "node-exporter"
     static_configs:
       - targets: ["10.210.2.117:9100"]
   ```

---

## 7. 📁 Directory Structure
```bash
~/containers/node-exporter/
├── docker-compose.yml
└── node_exporter_logs/
```

---

## 8. 🧹 Maintenance & Logs
Check logs if needed:
```bash
docker logs -f node-exporter
```

Restart Node Exporter:
```bash
docker-compose restart node_exporter
```

Remove container and volumes:
```bash
docker-compose down -v
```

---

## 9. 🕒 Auto Restart on Boot
Enable Docker service to ensure Node Exporter restarts automatically:
```bash
sudo systemctl enable docker
```

---

## 10. 🧠 Summary
| Component | Description |
|------------|--------------|
| **Container Name** | `node-exporter` |
| **Image** | `prom/node-exporter:latest` |
| **Port** | `9100` |
| **User** | `nobody (65534)` |
| **Network** | `prometheus_net` |
| **Log Path** | `~/containers/node-exporter/node_exporter_logs/` |

---
