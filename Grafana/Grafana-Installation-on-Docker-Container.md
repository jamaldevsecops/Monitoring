# 🟢 Deploy Grafana Using Docker and Docker Compose

## 1. 🎯 Objective
This SOP outlines the steps to deploy Grafana using Docker and Docker Compose on a Linux system. It covers creating a dedicated `devops` user, preparing directories, configuring Docker Compose, and running Grafana as a container.

## 2. 📋 Prerequisites
- 🖥️ A Linux server (RHEL, CentOS, Fedora, Ubuntu, or similar)  
- 🔑 Sudo/root access  
- 🌐 Network connectivity to download packages and Docker images  
- 🐳 Docker and Docker Compose installed  
- 🌉 Optional: External Docker network `grafana_net`

## 3. 🔄 Update System Packages
```bash
sudo dnf -y update   # For RHEL/CentOS/Fedora
# OR for Ubuntu/Debian
sudo apt update && sudo -y upgrade
```

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

## 7. 📂 Prepare Grafana Deployment Directory
```bash
mkdir -p ~/containers/grafana-server && cd ~/containers/grafana-server
mkdir grafana_data grafana_logs provisioning
```
Set proper ownership:
```bash
sudo chown -R 472:472 grafana_data grafana_logs provisioning
```
Create Docker network:
```bash
docker network create grafana_net
```

## 8. 📄 Create Docker Compose File for Grafana
Create `docker-compose.yml`:
```yaml
version: "3.9"

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana-server
    user: "472:472"
    ports:
      - "3000:3000"
    restart: always
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./grafana_data:/var/lib/grafana:z
      - ./grafana_logs:/var/log/grafana:z
      - ./provisioning:/etc/grafana/provisioning:ro,z
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 5
    mem_limit: 1024m
    cpus: 1.0
    networks:
      - grafana_net

networks:
  grafana_net:
    external: true
```

## 9. 🚀 Deploy Grafana Server
```bash
docker-compose up -d
docker ps
docker-compose ps
```

## 10. 📌 Post-Deployment Notes
- 🌐 Grafana URL: `http://<server_ip>:3000`  
- 🔑 Default login: User=`admin`, Password=`admin` (change after login)  
- 📄 Logs: `~/containers/grafana-server/grafana_logs/`  
- ⚙️ Provisioning configs in `provisioning/` folder

Optional: Check container health:
```bash
docker inspect -f '{{.State.Health.Status}}' grafana-server
```

