# 🏗️ Production-Grade Grafana Installation Guide (Docker Compose)

## 📋 Overview

This guide provides step-by-step instructions for deploying **Grafana** in a **production-grade Docker environment** using **Docker Compose**.  
It ensures **proper permissions, persistence, environment isolation**, and ease of maintenance — ideal for production, staging, or internal monitoring setups.

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
mkdir -p ~/containers/grafana/{data,config}
cd ~/containers/grafana
```

Your structure should look like this:

```
~/containers/grafana/
├── docker-compose.yml
├── config/
└── data/
```

---

## 🧾 Step 1: Verify Grafana Container User

Run the Grafana container interactively to check its default user and UID/GID:

```bash
docker run --rm -it grafana/grafana:latest sh
whoami
# Expected Output: grafana
id grafana
# Expected Output: uid=472(grafana) gid=472(grafana) groups=472(grafana)
exit
```

> This ensures we apply correct ownership to persistent directories.

---

## 🧾 Step 2: Set Proper Permissions

Apply ownership to the directories using the Grafana UID/GID:

```bash
chown -R 472:472 data config
```

> This ensures Grafana can read/write to `data` and `config` without permission issues.

---

## 🧾 Step 3: Create Docker Network

Create a dedicated Docker network for Grafana and related services:

```bash
docker network create grafana_net
```

---

## 🧾 Step 4: Create Docker Compose File

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
      - grafana_net

networks:
  grafana_net:
    external: true
```

### 🧠 Notes

- `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` define Grafana admin credentials.
- The `data` and `config` directories are mapped for **persistent storage**.
- The container restarts automatically on failure.
- The external `grafana_net` network allows you to connect **Prometheus**, **Loki**, etc.

---

## 🧾 Step 5: Start Grafana

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

## 🌐 Step 6: Access Grafana Dashboard

Open your browser and navigate to:

👉 **http://localhost:3000**

Login with the credentials defined in your environment (default: `admin / StrongPassword123!`).

---

## 🧰 Step 7: Configure Data Sources

Once logged in, configure your data sources:

1. Go to **Connections → Data Sources**
2. Click **Add Data Source**
3. Select from options like:
   - **Prometheus** (`http://prometheus:9090`)
   - **Loki** (`http://loki:3100`)
   - **MySQL / PostgreSQL / InfluxDB**

---

## 📦 Step 8: Persist and Backup

To make your deployment production-grade:

1. **Use named volumes** or mount to external storage (e.g., NFS, EBS, NAS).  
2. **Regularly back up** `~/containers/grafana/data`.  
3. **Version control** your configuration under `~/containers/grafana/config`.  

Example backup command:

```bash
tar -czvf grafana-backup-$(date +%F).tar.gz ~/containers/grafana/data
```

---

## 🧱 Step 9: Add Prometheus (Optional)

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
      - grafana_net
```

Then, configure Grafana to use `http://prometheus:9090` as a data source.

---

## 🔒 Step 10: Secure Your Deployment

For a production environment, ensure you:

- Use **strong admin credentials**
- Enable **HTTPS** with a reverse proxy (Nginx or Traefik)
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
| Grafana won’t start | Permission error | `chown -R 472:472 data config` |
| Login fails | Wrong password | Reset via `GF_SECURITY_ADMIN_PASSWORD` |
| Cannot reach Prometheus | Network misconfig | Ensure both containers share the `grafana_net` network |
| Data not persisting | Volume misconfig | Verify `./data` is mapped correctly |

---

## 🏁 Summary

With this setup, you now have a **production-ready Grafana** instance running in Docker Compose with:

- **Correct user permissions** for persistent data  
- **Dedicated Docker network** for isolation  
- **Persistent configuration and storage**  
- **Extensible architecture** (Prometheus, Loki, etc.)  
- Ready for **HTTPS, authentication, and alerting**  

---

## 🔗 References

- [Grafana Official Docs](https://grafana.com/docs/)
- [Grafana Docker Hub](https://hub.docker.com/r/grafana/grafana)
- [Grafana GitHub Repository](https://github.com/grafana/grafana)
- [Docker Compose Docs](https://docs.docker.com/compose/)
