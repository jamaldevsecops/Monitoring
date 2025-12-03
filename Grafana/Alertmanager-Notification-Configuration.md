# ✅ Prometheus + Alertmanager + Microsoft Teams Integration

## Overview
This guide explains how to integrate **Prometheus**, **Alertmanager**, and **Microsoft Teams** for alerting on CPU, Memory, and Disk usage using Node Exporter metrics.

---

## 🏗 Architecture
```
Node Exporter  ──► 📊 Prometheus  ──► 🚨 Alertmanager  ──► 💬 Microsoft Teams
```

---

## Steps

### 🔗 1. Create Microsoft Teams Webhook
- Use **Teams Workflows** to create an incoming webhook.
- Copy the webhook URL for later use.

### ⚙️ 2. Alertmanager Integration Option

#### Native Teams Receiver (Alertmanager >= 0.26)
Verify the Alertmanager configuration file `alertmanager.yml`.
```yaml
global:
  resolve_timeout: 1m

route:
  receiver: "default"   # fallback
  group_by: ["instance", "job"]
  group_wait: 30s # Wait before sending the first notification (to allow grouping).
  group_interval: 5m # Minimum time between notification for the same group.
  repeat_interval: 10m # How often to resent if the alert is still firing.

  # --- Child routes based on job name ---
  routes:
    # Linux job alerts
    - match:
        job: "DC1-ALL-LINUX-SERVERS"
      receiver: "msteams_linux"

    # Windows job alerts
    # - match:
    #     job: "DC1-ALL-WINDOWS-SERVERS"
    #   receiver: "msteams_windows"

    # Add more groups if needed
    # - match:
    #     job: "database-job"
    #   receiver: "msteams_database"


receivers:
  - name: "msteams_linux"
    msteamsv2_configs:
      - webhook_url: "https://teams-webhook-url-for-linux"
        send_resolved: true

#  - name: "msteams_windows"
#    msteamsv2_configs:
#      - webhook_url: "https://teams-webhook-url-for-windows"
#        send_resolved: true

  # fallback if no route matches
  - name: "default"
    msteamsv2_configs:
      - webhook_url: "https://fallback-webhook"
        send_resolved: true
```

### 📜 3. Prometheus Alert Rules Options: 1
Create `rules_node.yml` file under `~/prometheus/prometheus_config/` directory:
```yaml
groups:
  - name: LINUX-SERVERS-RESOURCE-UTILIZATION-ALERTS
    rules:
    # CPU Utilization Alert
    - alert: High-CPU-Usage
      expr: (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High CPU usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current CPU usage is {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High System Load (5m avg) Alert
    - alert: High-System-Load
      expr: (node_load5 * 100) / on(instance)(count(count(node_cpu_seconds_total) by (instance, cpu)) by (instance)) >= 85
      for: 5m
      labels:
        severity: '{{ if ge $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High system load (5m average) on {{ $labels.instance }}'
        description: '{{ if ge $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current Load average (5m) is {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High Memory Usage Alert
    - alert: High-Memory-Usage
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High memory usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current memory usage {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High Swap Usage Alert
    - alert: High-Swap-Usage
      expr: (node_memory_SwapTotal_bytes - node_memory_SwapFree_bytes) / node_memory_SwapTotal_bytes * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High swap usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current swap usage {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # Download Speed Alert 
    - alert: High-Network-Receive (RX)
      expr: max by (instance) (rate(node_network_receive_bytes_total[5m]) * 8 / 1000000) > 500
      for: 5m
      labels:
        severity: '{{ if gt $value 800.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network receive on {{ $labels.instance }}'
        description: '{{ if gt $value 800.0 }}Critical{{ else }}Warning{{ end }} alert for RX bandwidth on {{ $labels.instance }}. Current RX speed: {{ printf "%.2f" $value }} Mb/s. Warning threshold: 500 Mb/s, Critical threshold: 800 Mb/s.'

    # Upload Speed Alert
    - alert: High-Network-Transmit (TX)
      expr: max by (instance) (rate(node_network_transmit_bytes_total[5m]) * 8 / 1000000) > 500
      for: 5m
      labels:
        severity: '{{ if gt $value 800.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network transmit on {{ $labels.instance }}'
        description: '{{ if gt $value 800.0 }}Critical{{ else }}Warning{{ end }} alert for TX bandwidth on {{ $labels.instance }}. Current TX speed: {{ printf "%.2f" $value }} Mb/s. Warning threshold: 500 Mb/s, Critical threshold: 800 Mb/s.'

    # Disk Read Alert (If any instance reads more than 1500 (1.5GB) MB/s for 5 minutes, Trigger alert)
    - alert: High-Disk-Read
      expr: max by (instance) (rate(node_disk_read_bytes_total[5m]) / 1000000) > 1500
      for: 5m
      labels:
        severity: '{{ if gt $value 2000 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High disk read throughput on {{ $labels.instance }}'
        description: '{{ if gt $value 2000 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }}. Current read speed is {{ printf "%.2f" $value }} MB/s. Warning threshold: 1500 MB/s, Critical threshold: 2000 MB/s.'

    # Disk Write Alert (If any instance writes more than 300 MB/s for 5 minutes, Trigger alert)
    - alert: High-Disk-Write
      expr: max by (instance) (rate(node_disk_written_bytes_total[5m]) / 1000000) > 300
      for: 5m
      labels:
        severity: '{{ if gt $value 500 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High disk write throughput on {{ $labels.instance }}'
        description: '{{ if gt $value 500 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }}. Current write speed is {{ printf "%.2f" $value }} MB/s. Warning threshold: 300 MB/s, Critical threshold: 500 MB/s.'

    # Filesystem Space Alert
    - alert: FilesystemSpaceLow
      expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"})) * 100 > 80
      for: 15m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})'
        description: '{{ if gt $value 90.0 }}FS utilization > 90% (Critical){{ else }}FS utilization > 80% (Warning){{ end }} for 15m (current: {{ printf "%.2f" $value }}%).'
```
### 📜 3. Prometheus Alert Rules Options: 2
- All filesystems (except tmpfs, overlay, squashfs, devtmpfs):
  - Warning: >80% and ≤90%
  - Critical: >90%
- Only for /LOGS and /document:
  - Warning: >95% and ≤97%
  - Critical: >97%

Create `rules_node.yml` file under `~/prometheus/prometheus_config/` directory:
```yaml
groups:
  - name: LINUX-SERVERS-RESOURCE-UTILIZATION-ALERTS
    rules:
    # CPU Utilization Alert
    - alert: High-CPU-Usage
      expr: (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High CPU usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current CPU usage is {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High System Load (5m avg) Alert
    - alert: High-System-Load
      expr: (node_load5 * 100) / on(instance)(count(count(node_cpu_seconds_total) by (instance, cpu)) by (instance)) >= 85
      for: 5m
      labels:
        severity: '{{ if ge $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High system load (5m average) on {{ $labels.instance }}'
        description: '{{ if ge $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current Load average (5m) is {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High Memory Usage Alert
    - alert: High-Memory-Usage
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High memory usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current memory usage {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # High Swap Usage Alert
    - alert: High-Swap-Usage
      expr: (node_memory_SwapTotal_bytes - node_memory_SwapFree_bytes) / node_memory_SwapTotal_bytes * 100 > 85
      for: 5m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High swap usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }} in last 5 minutes. Current swap usage {{ printf "%.2f" $value }}%. Warning threshold: 85%, Critical threshold: 90%.'

    # Download Speed Alert 
    - alert: High-Network-Receive (RX)
      expr: max by (instance) (rate(node_network_receive_bytes_total[5m]) * 8 / 1000000) > 500
      for: 5m
      labels:
        severity: '{{ if gt $value 800.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network receive on {{ $labels.instance }}'
        description: '{{ if gt $value 800.0 }}Critical{{ else }}Warning{{ end }} alert for RX bandwidth on {{ $labels.instance }}. Current RX speed: {{ printf "%.2f" $value }} Mb/s. Warning threshold: 500 Mb/s, Critical threshold: 800 Mb/s.'

    # Upload Speed Alert
    - alert: High-Network-Transmit (TX)
      expr: max by (instance) (rate(node_network_transmit_bytes_total[5m]) * 8 / 1000000) > 500
      for: 5m
      labels:
        severity: '{{ if gt $value 800.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network transmit on {{ $labels.instance }}'
        description: '{{ if gt $value 800.0 }}Critical{{ else }}Warning{{ end }} alert for TX bandwidth on {{ $labels.instance }}. Current TX speed: {{ printf "%.2f" $value }} Mb/s. Warning threshold: 500 Mb/s, Critical threshold: 800 Mb/s.'

    # Disk Read Alert (If any instance reads more than 1500 (1.5GB) MB/s for 5 minutes, Trigger alert)
    - alert: High-Disk-Read
      expr: max by (instance) (rate(node_disk_read_bytes_total[5m]) / 1000000) > 1500
      for: 5m
      labels:
        severity: '{{ if gt $value 2000 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High disk read throughput on {{ $labels.instance }}'
        description: '{{ if gt $value 2000 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }}. Current read speed is {{ printf "%.2f" $value }} MB/s. Warning threshold: 1500 MB/s, Critical threshold: 2000 MB/s.'

    # Disk Write Alert (If any instance writes more than 300 MB/s for 5 minutes, Trigger alert)
    - alert: High-Disk-Write
      expr: max by (instance) (rate(node_disk_written_bytes_total[5m]) / 1000000) > 300
      for: 5m
      labels:
        severity: '{{ if gt $value 500 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High disk write throughput on {{ $labels.instance }}'
        description: '{{ if gt $value 500 }}Critical{{ else }}Warning{{ end }} alert on {{ $labels.instance }}. Current write speed is {{ printf "%.2f" $value }} MB/s. Warning threshold: 300 MB/s, Critical threshold: 500 MB/s.'

    # Filesystem Space Alert (LOGS/document have different thresholds)
    - alert: Low-Filesystem-Usage
      expr: |
        (
          (
            100 * (1 - (
              node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint=~"/LOGS|/document"}
              /
              node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint=~"/LOGS|/document"}
            ))
          ) > 97
        )
        OR
        (
          (
            100 * (1 - (
              node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint!~"/LOGS|/document"}
              /
              node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint!~"/LOGS|/document"}
            ))
          ) > 85
        )
      for: 1m

      labels:
        severity: warning

      annotations:
        summary: "Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})"
        description: 'Warning alert on instance {{ $labels.instance }} mountpoint {{ $labels.mountpoint }}. Current usage {{ printf "%.2f" $value }}% (Threshold: {{ if or (eq $labels.mountpoint "/LOGS") (eq $labels.mountpoint "/document") }}97%{{ else }}85%{{ end }}).'
```
---

### 🛠 4. Prometheus Config
Add to `prometheus.yml`:
```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "/etc/prometheus/rules_node.yml"
  # - "second_rules.yml"
```

---

### 🔄 5. Reload Configs
```bash
curl -X POST http://<prometheus-host>:9090/-/reload
curl -X POST http://<alertmanager-host>:9093/-/reload
```
or, 
```bash
docker-compose restart alertmanager
docker-compose restart prometheues
```
On Prometheus:
<img width="1662" height="356" alt="image" src="https://github.com/user-attachments/assets/a29b3c23-42d1-4dd7-8c7d-fef00bd3c7ca" />

---

## Best Practices
- Use Workflows webhooks for Teams.
- Keep secrets out of Git.
- Filter pseudo filesystems in disk alerts.
- Test with Watchdog alert.

---

## File Placement
```
prometheus/
  ├─ prometheus_config/
      ├─ prometheus.yml
      └─ rules_node.yml
alert-manager/
  ├─ alertmanager_config/
      └─ alertmanager.yml
```
