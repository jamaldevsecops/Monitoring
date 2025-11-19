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
  receiver: "msteams"
  group_by: ["alertname", "instance"]
  group_wait: 30s # Wait before sending the first notification (to allow grouping).
  group_interval: 5m # Minimum time between notification for the same group.
  repeat_interval: 3h # How often to resent if the alert is still firing.

receivers:
  - name: "msteams"
    msteamsv2_configs:
      - webhook_url: "https://<teams-webhook-url>"
        send_resolved: true
```

### 📜 3. Prometheus Alert Rules Options: 1
Create `rules_node.yml` file under `~/prometheus/prometheus_config/` directory:
```yaml
groups:
  - name: node_resource_alerts
    rules:
    # CPU Utilization Alert
    - alert: HighCPUUtilization
      expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High CPU usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}CPU > 90% (Critical){{ else }}CPU > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'

    # Memory Utilization Alert
    - alert: HighMemoryUtilization
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High memory usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Mem utilization > 90% (Critical){{ else }}Mem utilization > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'

    # Filesystem Space Alert
    - alert: FilesystemSpaceLow
      expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs"})) * 100 > 80
      for: 15m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})'
        description: '{{ if gt $value 90.0 }}FS utilization > 90% (Critical){{ else }}FS utilization > 80% (Warning){{ end }} for 15m (current: {{ printf "%.2f" $value }}%).'

    # Network Bandwidth Utilization Alert
    - alert: HighNetworkBandwidthUtilization
      expr: ((rate(node_network_receive_bytes_total[5m]) + rate(node_network_transmit_bytes_total[5m])) / node_network_speed_bytes) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network bandwidth usage on {{ $labels.instance }} ({{ $labels.device }})'
        description: '{{ if gt $value 90.0 }}Network utilization > 90% (Critical){{ else }}Network utilization > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'
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
  - name: node_resource_alerts
    rules:
    # CPU Utilization Alert
    - alert: HighCPUUtilization
      expr: avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High CPU usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}CPU > 90% (Critical){{ else }}CPU > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'

    # Memory Utilization Alert
    - alert: HighMemoryUtilization
      expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High memory usage on {{ $labels.instance }}'
        description: '{{ if gt $value 90.0 }}Mem utilization > 90% (Critical){{ else }}Mem utilization > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'

    # Network Bandwidth Utilization Alert
    - alert: HighNetworkBandwidthUtilization
      expr: ((rate(node_network_receive_bytes_total[5m]) + rate(node_network_transmit_bytes_total[5m])) / node_network_speed_bytes) * 100 > 80
      for: 10m
      labels:
        severity: '{{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}'
      annotations:
        summary: 'High network bandwidth usage on {{ $labels.instance }} ({{ $labels.device }})'
        description: '{{ if gt $value 90.0 }}Network utilization > 90% (Critical){{ else }}Network utilization > 80% (Warning){{ end }} for 10m (current: {{ printf "%.2f" $value }}%).'


    # Filesystem Space Alert (LOGS/document have different thresholds)
    - alert: FilesystemSpaceLow
      expr: |
        (
          # for /LOGS and /document: usage >95%
          (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint=~"/LOGS|/document"} 
                / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint=~"/LOGS|/document"})) * 100 > 95
        )
        OR
        (
          # for all others: usage >80%
          (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint!~"/LOGS|/document"} 
                / node_filesystem_size_bytes{fstype!~"tmpfs|overlay|squashfs|devtmpfs", mountpoint!~"/LOGS|/document"})) * 100 > 80
        )
      for: 15m

      labels:
        severity: >
          {{ if or (eq $labels.mountpoint "/LOGS") (eq $labels.mountpoint "/document") }}
            {{ if gt $value 97.0 }}critical{{ else }}warning{{ end }}
          {{ else }}
            {{ if gt $value 90.0 }}critical{{ else }}warning{{ end }}
          {{ end }}

      annotations:
        summary: "Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})"
        description: >
          {{ if or (eq $labels.mountpoint "/LOGS") (eq $labels.mountpoint "/document") }}
            {{ if gt $value 97.0 }}
              FS utilization > 97% (Critical)
            {{ else }}
              FS utilization > 95% (Warning)
            {{ end }}
          {{ else }}
            {{ if gt $value 90.0 }}
              FS utilization > 90% (Critical)
            {{ else }}
              FS utilization > 80% (Warning)
            {{ end }}
          {{ end }}
          for 15m (current: {{ printf "%.2f" $value }}%).
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
