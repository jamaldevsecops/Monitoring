# 📘 SOP: Deploy cAdvisor, Integrate with Prometheus & Import Grafana Dashboard 13946

## 📌 1. Purpose  
This SOP explains how to:  
- 🚀 Deploy **cAdvisor** using Docker  
- 📊 Integrate cAdvisor with **Prometheus**  
- 📈 Import Grafana Dashboard **13946** for visualization  

---

## 🧰 2. Prerequisites

### 🖥️ System Requirements
- Linux host  
- Docker & Docker Compose installed  
- Internet access to pull container images  

### 📦 Installed Components
- Docker  
- Prometheus  
- Grafana  

### 🌐 Network Ports
| Service      | Port |
|--------------|------|
| cAdvisor     | 8080 |
| Prometheus   | 9090 |
| Grafana      | 3000 |

---

## 🏁 3. Procedure

---

## Step 1️⃣ — Deploy cAdvisor

### Create cAdvisor service
```yaml
services:
  cadvisor:
    image: gcr.io/google-containers/cadvisor:latest
    container_name: cadvisor
    restart: always
    ports:
      - "192.168.20.126:8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    deploy:
      resources:
        limits:
          memory: 1024M
          cpus: "0.25"
          pids: 100

    security_opt:
      - no-new-privileges:true
      - label:type:container_t # [For Fedora Based OS Only]
```

### Start cAdvisor
```
docker compose up -d cadvisor
```

### Validate
```
curl http://localhost:8080/metrics | head
```

---

## Step 2️⃣ — Add cAdvisor to Prometheus

### Edit `prometheus.yml`
```yaml
scrape_configs:
  - job_name: cadvisor
    metrics_path: /metrics
    static_configs:
      - targets: ['cadvisor:8080']
```

### Restart Prometheus
```
docker compose restart prometheus
```

### Verify Target  
Visit:  
👉 `http://<prometheus-host>:9090/targets`

---

## Step 3️⃣ — Add Prometheus to Grafana

### Configure Data Source  
1. ⚙️ Grafana → Configuration → Data Sources  
2. ➕ Add Data Source  
3. Choose **Prometheus**  
4. Set URL:  
   - `http://prometheus:9090` (same docker network)  
   - or external IP  
5. ✅ Save & Test  

---

## Step 4️⃣ — Import Grafana Dashboard

### Import  
1. 📄 Dashboards → Import  
2. Enter ID: **13946**  
3. Select Prometheus as data source  
4. ✔ Import  

---

## 🧪 5. Validation Checklist

| Task | Status |
|------|--------|
| cAdvisor running | ☐ |
| Prometheus scraping cAdvisor | ☐ |
| Target = UP | ☐ |
| Grafana datasource working | ☐ |
| Dashboard imported | ☐ |
| Metrics visible | ☐ |

---

## 🛠️ 6. Troubleshooting

### ❗ cAdvisor metrics missing
- Validate required volumes  
- Check Docker / SELinux permissions  

### ❗ Prometheus shows DOWN  
- Ensure containers share same Docker network  

### ❗ Grafana cannot read Prometheus  
- Verify data source URL  
- Check Prometheus logs  

---

## ✅ 7. Conclusion  
This SOP sets up a fully functional monitoring pipeline using:  
- 📦 cAdvisor  
- 📊 Prometheus  
- 📈 Grafana Dashboard 13946  

