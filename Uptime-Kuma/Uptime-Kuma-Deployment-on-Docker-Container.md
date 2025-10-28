# 🚀 Uptime Kuma Deployment using Docker Compose

This SOP provides step-by-step instructions to deploy **Uptime Kuma**, a self-hosted monitoring tool, using Docker Compose.

---

## 📦 Prerequisites

Before you begin, make sure you have the following installed on your system:

- 🐳 **Docker**
- 🧩 **Docker Compose**
- 🔐 Environment variables file (`.env`) with the following values defined:
  ```bash
  ADMIN_EMAIL=you@example.com
  ADMIN_PASSWORD=YourStrongPassword
  ```

---

## 🗂️ Folder Structure

```bash
uptime-kuma/
├── docker-compose.yml
├── .env
└── uptimekuma_data/
```

> 📝 The `uptimekuma_data` directory stores persistent data for Uptime Kuma.

---

## ⚙️ Docker Compose Configuration

Below is the complete **docker-compose.yml** configuration for deploying **Uptime Kuma**:

```yaml
version: "3.9"

services:
  uptime-kuma:
    image: elestio/uptime-kuma:latest
    container_name: dc1-uptime-kuma-core
    restart: always
    environment:
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
      HTTP_PROXY: http://10.210.2.200:4680
      HTTPS_PROXY: http://10.210.2.200:4680
      NO_PROXY: localhost,127.0.0.1

    volumes:
      - ./uptimekuma_data:/app/data:z
    ports:
      - "10.210.2.112:30002:3001"
    networks:
      - monitoring
    deploy:
      resources:
        limits:
          memory: 2048M
          cpus: "1"
          pids: 100
    security_opt:
      - no-new-privileges:true
      - label:type:container_t
    read_only: true

networks:
  monitoring:
    external: true
```

---

## ▶️ Deployment Steps

### 🏗️ 1. Create Project Directory
```bash
mkdir -p ~/uptime-kuma && cd ~/uptime-kuma
```

### 🧩 2. Create `.env` File
```bash
cat <<EOF > .env
ADMIN_EMAIL=you@example.com
ADMIN_PASSWORD=YourStrongPassword
EOF
```

### 🧱 3. Create Data Directory
```bash
mkdir -p uptimekuma_data
```

### 🧾 4. Create `docker-compose.yml`
Copy the YAML configuration above and paste it into this file.

### 🚀 5. Deploy the Stack
```bash
docker compose up -d
```

### 🧠 6. Verify Deployment
```bash
docker ps | grep uptime-kuma
```

Then open your browser and navigate to:

👉 `http://10.210.2.112:30002`

Login using the credentials from your `.env` file.

---

## 🧹 Maintenance

### 🔁 Restart the Container
```bash
docker compose restart uptime-kuma
```

### 📜 View Logs
```bash
docker compose logs -f uptime-kuma
```

### 💾 Backup Data
```bash
tar -czvf uptimekuma_backup_$(date +%F).tar.gz uptimekuma_data/
```

### 🗑️ Remove Everything (if needed)
```bash
docker compose down -v
```

---

## ⚙️ Resource Configuration

| Resource | Limit |
|-----------|--------|
| 🧠 Memory | 2048 MB |
| 🧮 CPU | 1 Core |
| 🧍 PIDs | 100 |

> 🧱 The configuration uses SELinux and Docker security options (`no-new-privileges` and `read_only`) for extra protection.

---

## 🛡️ Security Recommendations

✅ Use strong admin credentials in your `.env` file  
✅ Run Uptime Kuma behind a reverse proxy (e.g., Nginx or Traefik)  
✅ Enable HTTPS if running in production  
✅ Regularly back up the `/app/data` volume

---

## 🎯 Summary

| Component | Description |
|------------|-------------|
| 🧩 **Image** | elestio/uptime-kuma:latest |
| 🧠 **Purpose** | Self-hosted uptime monitoring tool |
| 🌐 **Port Mapping** | 10.210.2.112:30002 → 3001 |
| 💾 **Volume** | ./uptimekuma_data → /app/data |
| 🔒 **Security** | Read-only container, SELinux enabled |

---

### ✅ Deployment Complete!

Uptime Kuma should now be accessible at:  
👉 **http://10.210.2.112:30002**

Enjoy your self-hosted uptime monitoring dashboard!
