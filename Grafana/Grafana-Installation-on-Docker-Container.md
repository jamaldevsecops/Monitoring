# 🏗️ Production-Grade Grafana Installation Guide (Docker Compose)

## 📋 Overview

This guide provides step-by-step instructions for deploying **Grafana** in a **production-grade Docker environment** using **Docker Compose**.  
It ensures persistence, environment isolation, and ease of maintenance — ideal for production, staging, or internal monitoring setups.

---

## ⚙️ Prerequisites

Before proceeding, make sure you have:

- 🐳 **Docker** (v20+)
- ⚙️ **Docker Compose** (v2+)
- 🌐 Internet access (for pulling images)
- 💾 Sufficient disk space for persistent storage

---

## 📁 Directory Structure

Create a working directory for Grafana setup:

```bash
mkdir -p /opt/grafana/{data,config}
cd /opt/grafana
```

Your structure should look like this:

```
/opt/grafana/
├── docker-compose.yml
├── config/
└── data/
```

---

## 🧾 Step 1: Create Docker Compose File

Create a file named **docker-compose.yml**:

```yaml
version: '3.9'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=StrongPassword123!
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SERVER_ROOT_URL=http://localhost:3000
    volumes:
      - ./data:/var/lib/grafana
      - ./config:/etc/grafana
    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge
```

### 🧠 Notes
- `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` define Grafana admin credentials.
- The `data` directory is mapped for **persistent storage**.
- The container restarts automatically on failure.
- The `monitoring` network allows you to connect **Prometheus** or **Loki** containers later.

---

## 🧾 Step 2: Set Proper Permissions

Ensure Grafana can write to its local storage:

```bash
sudo chown -R 472:472 data
```

> **Note:** UID `472` is the default Grafana user inside the container.

---

## 🧾 Step 3: Start Grafana

Run the following command to start Grafana:

```bash
docker-compose up -d
```

Check the running container:

```bash
docker ps
```

You should see output similar to:

```
CONTAINER ID   IMAGE                 STATUS         PORTS
abc12345def6   grafana/grafana:latest   Up 10s     0.0.0.0:3000->3000/tcp
```

---

## 🌐 Step 4: Access Grafana Dashboard

Open your browser and navigate to:

👉 **http://localhost:3000**

Login with the credentials defined in your environment (default: `admin / StrongPassword123!`).

---

## 🧰 Step 5: Configure Data Sources

Once logged in, configure your data sources:

1. Go to **Connections → Data Sources**
2. Click **Add Data Source**
3. Select from options like:
   - **Prometheus** (`http://prometheus:9090`)
   - **Loki** (`http://loki:3100`)
   - **MySQL / PostgreSQL / InfluxDB**

---

## 📦 Step 6: Persist and Backup

To make your deployment production-grade:

1. **Use named volumes** or mount to an external storage (e.g., NFS, EBS, or NAS).  
2. **Regularly back up** `/opt/grafana/data`.  
3. **Version control** your configuration under `/opt/grafana/config`.  

Example backup command:

```bash
tar -czvf grafana-backup-$(date +%F).tar.gz /opt/grafana/data
```

---

## 🧱 Step 7: Add Prometheus (Optional)

Extend your stack by adding **Prometheus** for metrics collection:

```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - monitoring
```

Then, configure Grafana to use `http://prometheus:9090` as a data source.

---

## 🔒 Step 8: Secure Your Deployment

For a production environment, ensure you:

- Use **strong admin credentials**
- Enable **HTTPS** with reverse proxy (Nginx or Traefik)
- Configure **LDAP or OAuth** for authentication
- Enable **alerting** and **notifications**
- Restrict **inbound ports** using a firewall

Example Nginx reverse proxy snippet:

```nginx
server {
    listen 443 ssl;
    server_name grafana.example.com;

    ssl_certificate /etc/ssl/certs/fullchain.pem;
    ssl_certificate_key /etc/ssl/private/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🧩 Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|-----------|
| Grafana won’t start | Permission error | `sudo chown -R 472:472 data` |
| Login fails | Wrong password | Reset via `GF_SECURITY_ADMIN_PASSWORD` |
| Cannot reach Prometheus | Network misconfig | Ensure both containers share the `monitoring` network |
| Data not persisting | Volume misconfig | Verify `./data` is mapped correctly |

---

## 🧠 Summary

With this setup, you now have a **production-ready Grafana** instance running in Docker Compose with:

- Persistent data storage  
- Configurable environment variables  
- Network isolation  
- Easy extensibility (Prometheus, Loki, etc.)  
- Ready for HTTPS and reverse proxy

---

## 🔗 References

- [Grafana Official Docs](https://grafana.com/docs/)
- [Grafana Docker Hub](https://hub.docker.com/r/grafana/grafana)
- [Grafana GitHub Repository](https://github.com/grafana/grafana)
- [Docker Compose Docs](https://docs.docker.com/compose/)
