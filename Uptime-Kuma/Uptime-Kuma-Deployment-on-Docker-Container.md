# 🟢 Deploy Uptime Kuma Using Docker and Docker Compose

## 1. 🎯 Objective
This SOP outlines the steps to deploy **Uptime Kuma** using Docker and Docker Compose on a Linux system.  
It includes directory preparation, permissions setup, environment configuration, and container deployment.

---

## 2. 📋 Prerequisites
- 🖥️ A Linux server (RHEL, CentOS, Fedora, Ubuntu, or similar)  
- 🔑 Sudo/root access  
- 🌐 Internet connectivity to pull Docker images  
- 🐳 Docker and Docker Compose installed  
- 🌉 External Docker network: `monitoring` (will be created if not present)

---

## 3. 🔄 Update System Packages
```bash
sudo dnf -y update   # For RHEL/CentOS/Fedora
# OR for Ubuntu/Debian
sudo apt update && sudo -y upgrade
```

---

## 4. 👤 Create DevOps User (if not already present)
```bash
sudo useradd devops
sudo usermod -aG docker devops
newgrp docker
```

---

## 5. 📂 Directory Setup and Permissions
Create and set up the directory structure for Uptime Kuma:
```bash
mkdir -p ~/containers/uptime-kuma && cd ~/containers/uptime-kuma
mkdir uptimekuma_data uptimekuma_logs
```

Determine the container’s running user:
```bash
docker run --rm -it --entrypoint /bin/sh elestio/uptime-kuma:latest
/ $ id
uid=1000(node) gid=1000(node)
```

Assign correct ownership:
```bash
sudo chown -R 1000:1000 uptimekuma_data uptimekuma_logs
```

(Optional) Create Docker network if not already available:
```bash
docker network create monitoring
```

---

## 6. ⚙️ Environment Configuration
Create a `.env` file in the same directory with the following variables:
```bash
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=StrongPassword123!
```

---

## 7. 📄 Create Docker Compose File
Create `docker-compose.yml` under `~/containers/uptime-kuma/`:
```yaml
services:
  uptime-kuma:
    image: elestio/uptime-kuma:latest
    container_name: uptime-kuma
    restart: always
    environment:
      ADMIN_EMAIL: ${ADMIN_EMAIL}
      ADMIN_PASSWORD: ${ADMIN_PASSWORD}
      HTTP_PROXY: http://10.210.2.200:4680
      HTTPS_PROXY: http://10.210.2.200:4680
      NO_PROXY: localhost,127.0.0.1
    volumes:
      - ./uptimekuma_data:/app/data:z
      - ./uptimekuma_logs:/app/logs:z
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

## 8. 🚀 Deploy Uptime Kuma
Run the container:
```bash
docker-compose up -d
```

Verify status:
```bash
docker ps
docker-compose ps
```

---

## 9. 🧩 Post-Deployment Notes
- 🌐 Web Interface: `http://10.210.2.112:30002`  
- 🔑 Admin Login: use credentials from `.env` file  
- 📄 Data Directory: `~/containers/uptime-kuma/uptimekuma_data/`  
- 🪵 Logs Directory: `~/containers/uptime-kuma/uptimekuma_logs/`  

Check container logs:
```bash
docker logs -f uptime-kuma
```

Health check:
```bash
curl -I http://10.210.2.112:30002
```

Restart container:
```bash
docker-compose restart
```

---

## 10. 🧹 Maintenance Tips
- Backup the `uptimekuma_data` folder regularly.  
- Update the container image monthly:
  ```bash
  docker-compose pull
  docker-compose up -d
  ```
- Check disk usage:
  ```bash
  du -sh ~/containers/uptime-kuma/uptimekuma_data
  ```

---

## ✅ Summary
This guide provides a production-ready setup for **Uptime Kuma** using Docker and Docker Compose with:
- Proper directory structure  
- Secure file permissions  
- Proxy and environment configuration  
- Resource limits and security hardening

---
