# 🧭 **Deploying Node Exporter on Linux Using Binary**
📅 **Date:** 2025-11-10  

---

### 📘 Overview
This SOP outlines the procedure for deploying **Node Exporter** on a Linux system using its **binary distribution**. Node Exporter is a Prometheus exporter that provides system metrics such as CPU, memory, disk, and network usage. This guide also includes instructions for setting up Node Exporter as a **systemd service**.

---

## ⚙️ Step 1: Download Node Exporter Binary

1. Visit the [Node Exporter Downloads](https://prometheus.io/download/#node_exporter) page and download the latest **Linux AMD64** tarball.
2. Run the following commands:

```bash
cd /opt
sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```

If using a proxy:

```bash
wget -e use_proxy=yes -e http_proxy=http://10.210.10.173:4899 \     
  -e https_proxy=http://10.210.10.173:4899 \     
  https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
```

Extract and move files:

```bash
sudo tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
sudo mv node_exporter-1.8.2.linux-amd64 node_exporter
```

---

## 👤 Step 2: Create Node Exporter User

```bash
sudo useradd --no-create-home --shell /bin/false nodeusr
```

---

## 📂 Step 3: Set Up Directories and Permissions

Move the binary and set ownership:

```bash
sudo cp /opt/node_exporter/node_exporter /usr/local/bin/
sudo chown nodeusr:nodeusr /usr/local/bin/node_exporter
```

---

## 🧾 Step 4: Create systemd Unit File

Create and open the file:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste the following content:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter \  
  --collector.logind \  
  --collector.processes \  
  --collector.systemd \  
  --collector.tcpstat

[Install]
WantedBy=multi-user.target
```

---

## 🔁 Step 5: Reload systemd and Start Node Exporter

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

---

## 🌐 Step 6: Verify Node Exporter Metrics

Open your browser or use `curl` to check the metrics endpoint:

👉 `http://<your-server-ip>:9100/metrics`

```bash
curl http://localhost:9100/metrics
```

If successful, you’ll see metrics output including CPU, memory, and filesystem statistics.

---

### 🧩 Notes
- Default port: **9100**
- Binary path: `/usr/local/bin/node_exporter`
- Service file: `/etc/systemd/system/node_exporter.service`
- To view logs: `sudo journalctl -u node_exporter -f`
- Node Exporter is a recommended companion to **Prometheus** for system-level monitoring.
