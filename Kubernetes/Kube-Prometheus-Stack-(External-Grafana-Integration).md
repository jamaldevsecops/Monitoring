# 🌐 **Kube-Prometheus-Stack Deployment (External Grafana Integration)**

This SOP describes how to deploy **Kube-Prometheus-Stack** in Kubernetes while using an **external Grafana instance** for visualization.

---

## 🧩 **0) Prerequisites**
- ✅ Running Kubernetes cluster (v1.21+)
- ✅ `kubectl` and `helm` installed
- ✅ Ingress Controller (e.g., NGINX)
- ✅ DNS records for:
  - `prometheusx.apsis.localnet`
  - `alertmanager.apsis.localnet`
- ✅ External Grafana accessible (e.g. `grafana.apsis.localnet`)
- ✅ Internal DNS points to ingress controller IP: **192.168.20.162**

---

## ⚙️ **1) Create Namespace**
```bash
kubectl create namespace monitoring
```

---

## 🧾 **2) Create `values.yaml`**
Create a minimal file at `~/kube-prometheus-stack/values.yaml`:

```yaml
grafana:
  enabled: false  # Disable internal Grafana

prometheus:
  service:
    type: ClusterIP
  ingress:
    enabled: true
    ingressClassName: nginx
    hosts:
      - prometheusx.apsis.localnet
    path: /
    pathType: Prefix
  prometheusSpec:
    retention: 15d
    scrapeInterval: 30s

alertmanager:
  ingress:
    enabled: true
    ingressClassName: nginx
    hosts:
      - alertmanager.apsis.localnet
    path: /
    pathType: Prefix
  alertmanagerSpec:
    replicas: 1

nodeExporter:
  enabled: true

kubeStateMetrics:
  enabled: true
```

---

## 🚀 **3) Deploy Using Helm**
Using the OCI chart:
```bash
helm install kube-prometheus-stack \
  oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack \
  --version 79.4.0 \
  -n monitoring \
  -f values.yaml
```

Or using Helm repo:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values.yaml
```

---

## 🔍 **4) Verify Deployment**
```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```

You should only see **Prometheus** and **Alertmanager** running (no Grafana).

Access in browser:
- 📊 Prometheus → http://prometheusx.apsis.localnet
- 🚨 Alertmanager → http://alertmanager.apsis.localnet

---

# 🧠 **5) Connect External Grafana to Prometheus**

### A) **From External Grafana UI**
1. Open Grafana (e.g., http://grafana.apsis.localnet)
2. Go to **Connections → Data sources → Add data source**
3. Choose **Prometheus**
4. Set URL:
   ```
   http://prometheusx.apsis.localnet
   ```
5. Save & Test → ✅ “Data source is working”

### B) **If Grafana is inside the same cluster**
Use:
```
http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090
```

---

# 📈 **6) Import Dashboards (Recommended)**
From [Grafana.com Dashboards](https://grafana.com/grafana/dashboards):

| Dashboard Name | ID | Description |
|----------------|----|--------------|
| 🧩 Kubernetes / Compute Resources / Cluster | 315 | Cluster-level metrics |
| 🧩 Node Exporter Full | 1860 | Node metrics |
| 🧩 Kubernetes / API Server | 12006 | API Server stats |

Import them under **Dashboards → Import → Enter ID**.

---

# 🧱 **7) Secure Prometheus Access (Optional)**
To restrict access to Prometheus UI:
- Enable **Basic Auth** or mTLS via Ingress annotations.
- Or restrict by internal network/firewall rules.

Example basic auth annotation (NGINX Ingress):
```yaml
annotations:
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: prometheus-auth
  nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
```

---

# 🧼 **8) Cleanup / Disable Grafana Later**
If Grafana was already deployed earlier and needs removal:
```bash
helm upgrade kube-prometheus-stack \
  oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack \
  -n monitoring \
  -f values.yaml --set grafana.enabled=false

kubectl delete deployment kube-prometheus-stack-grafana -n monitoring
kubectl delete svc kube-prometheus-stack-grafana -n monitoring
kubectl delete ingress kube-prometheus-stack-grafana -n monitoring
```

---

✅ **Result:**  
Prometheus and Alertmanager are deployed in-cluster, and your **external Grafana** visualizes cluster metrics seamlessly through Prometheus.
