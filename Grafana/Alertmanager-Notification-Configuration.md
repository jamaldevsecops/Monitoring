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
  receiver: "teams"
  group_by: ["alertname", "instance"]
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h

receivers:
  - name: "teams"
    msteamsv2_configs:
      - webhook_url: "https://<teams-webhook-url>"
        send_resolved: true
```

### 📜 3. Prometheus Alert Rules
Create `rules_node.yml` file under `~/prometheus/prometheus_config/` directory:
```yaml
groups:
  - name: node_resource_alerts
    rules:
    - alert: HighCPUUtilization
      expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100 > 85
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "High CPU usage on {{ $labels.instance }}"
        description: "CPU > 85% for 10m (current: {{ printf \"%.2f\" $value }}%)."

    - alert: HighMemoryUtilization
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
      for: 10m
      labels:
        severity: warning
      annotations:
        summary: "High memory usage on {{ $labels.instance }}"
        description: "Mem utilization > 85% for 10m (current: {{ printf \"%.2f\" $value }}%)."

    - alert: FilesystemSpaceLow
      expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"})) * 100 > 85
      for: 15m
      labels:
        severity: warning
      annotations:
        summary: "Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})"
        description: "FS utilization > 85% for 15m (current: {{ printf \"%.2f\" $value }}%)."
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
