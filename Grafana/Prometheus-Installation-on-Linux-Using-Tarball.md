# 🧭 **Deploying Prometheus on Linux Using Binary**

---

### 📘 Overview
This SOP outlines the procedure for deploying **Prometheus** on a Linux system using its **binary distribution**. It includes instructions for configuring Prometheus as a **systemd service** and setting up **data retention policies**.

---

## ⚙️ Step 1: Download Prometheus Binary

1. Navigate to [Prometheus Downloads](https://prometheus.io/download/) and download the latest **Linux AMD64** tarball.
2. Run the following commands:

```bash
cd /opt
sudo wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```

If using a proxy:

```bash
wget -e use_proxy=yes -e http_proxy=http://10.210.10.173:4899      -e https_proxy=http://10.210.10.173:4899      https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```

Extract and move files:

```bash
sudo tar -xvf prometheus-3.5.0.linux-amd64.tar.gz
sudo mv prometheus-3.5.0.linux-amd64 prometheus
```

---

## 👤 Step 2: Create Prometheus User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

---

## 📂 Step 3: Set Up Directories and Permissions

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo cp /opt/prometheus/prometheus.yml /etc/prometheus/
sudo cp /opt/prometheus/prometheus /usr/local/bin/
sudo cp /opt/prometheus/promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

---

## 🧾 Step 4: Create systemd Unit File

Create and open the file:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste the following content:

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus   --config.file=/etc/prometheus/prometheus.yml   --storage.tsdb.path=/var/lib/prometheus/   --storage.tsdb.retention.time=30d   --web.console.templates=/opt/prometheus/consoles   --web.console.libraries=/opt/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

---

## 🔁 Step 5: Reload systemd and Start Prometheus

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

---

## 🌐 Step 6: Access Prometheus Web UI

Open your browser and navigate to:

👉 `http://<your-server-ip>:9090`

---

### 🧩 Notes
- Default port: **9090**
- Configuration file: `/etc/prometheus/prometheus.yml`
- Data path: `/var/lib/prometheus`
- To view logs: `sudo journalctl -u prometheus -f`
