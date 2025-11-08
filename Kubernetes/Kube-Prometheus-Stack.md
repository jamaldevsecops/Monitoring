# 🚀 **Kube-Prometheus-Stack Deployment**
For internal domain: **apsis.localnet**

---

## 🧩 **0) Prerequisites**
- ✅ Kubernetes cluster (v1.21+)
- ✅ `kubectl` and `helm` installed
- ✅ Ingress Controller (NGINX or equivalent)
- ✅ DNS for:
  - `grafanax.apsis.localnet`
  - `prometheusx.apsis.localnet`
  - `alertmanagerx.apsis.localnet`
  → All pointing to **192.168.20.162**

---

## ⚙️ **1) Create Namespace**
```bash
kubectl create namespace monitoring
```

---

## 🧾 **2) Prepare `values.yaml`**
Create file at `~/kube-prometheus-stack/values.yaml`:

```yaml
# --- Grafana ---
grafana:
  adminUser: admin
  adminPassword: "ChangeMeNow!"   # set a strong internal password
  service:
    type: ClusterIP
  ingress:
    enabled: true
    ingressClassName: nginx       # or your class name
    hosts:
      - grafanax.apsis.localnet
    path: /
    pathType: Prefix
    tls: []                       # add TLS block if you have internal CA/secret

# --- Prometheus ---
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
    tls: []
  prometheusSpec:
    retention: 15d
    scrapeInterval: 30s

# --- Alertmanager ---
alertmanager:
  service:
    type: ClusterIP
  ingress:
    enabled: true
    ingressClassName: nginx
    hosts:
      - alertmanagerx.apsis.localnet
    path: /
    pathType: Prefix
    tls: []

# Reduce noise: only if you want fewer example rules/targets
defaultRules:
  create: true

# Optional: Persistent storage (uncomment & tailor)
# prometheus:
#   prometheusSpec:
#     storageSpec:
#       volumeClaimTemplate:
#         spec:
#           accessModes: ["ReadWriteOnce"]
#           resources:
#             requests:
#               storage: 50Gi
# grafana:
#   persistence:
#     enabled: true
#     size: 10Gi
# alertmanager:
#   alertmanagerSpec:
#     storage:
#       volumeClaimTemplate:
#         spec:
#           accessModes: ["ReadWriteOnce"]
#           resources:
#             requests:
#               storage: 10Gi
```

---

## 🚀 **3) Install Chart**
```bash
helm upgrade kube-prometheus-stack \
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
  -n monitoring -f values.yaml
```

---

## 🔍 **4) Verify**
```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get ingress -n monitoring
```

Access via browser:
- 🌐 Grafana → http://grafanax.apsis.localnet  
- 📊 Prometheus → http://prometheusx.apsis.localnet  
- 🚨 Alertmanager → http://alertmanagerx.apsis.localnet

---

# 🔄 **Apply Configuration Changes (values.yaml Updates)**

## ✏️ **1) Edit the values file**
```bash
nano ~/kube-prometheus-stack/values.yaml
```

## ⚡ **2) Apply updated configuration**
If deployed via OCI:
```bash
helm upgrade kube-prometheus-stack \
  oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack \
  -n monitoring -f values.yaml
```
Or if using the Helm repo:
```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring -f values.yaml
```

## 🔍 **3) Verify rollout**
```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
helm status kube-prometheus-stack -n monitoring
```

## 🧠 **4) Dry-run test (preview without applying)**
```bash
helm upgrade kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n monitoring -f values.yaml --dry-run --debug
```

## ⏪ **5) Rollback if needed**
List revisions:
```bash
helm history kube-prometheus-stack -n monitoring
```
Rollback to a previous version:
```bash
helm rollback kube-prometheus-stack <REVISION_NUMBER> -n monitoring
```

---

# 🧼 **Uninstallation / Cleanup Steps**

## 🗑️ **A) Uninstall Helm Release**
```bash
helm uninstall kube-prometheus-stack -n monitoring
```

## 📦 **B) Remove PVCs**
```bash
kubectl get pvc -n monitoring
kubectl delete pvc -n monitoring -l app.kubernetes.io/instance=kube-prometheus-stack
```

## 🧱 **C) Remove CRDs**
```bash
kubectl get crds | grep monitoring.coreos.com
kubectl delete crd alertmanagers.monitoring.coreos.com \
  alertmanagerconfigs.monitoring.coreos.com \
  podmonitors.monitoring.coreos.com \
  probes.monitoring.coreos.com \
  prometheuses.monitoring.coreos.com \
  prometheusrules.monitoring.coreos.com \
  servicemonitors.monitoring.coreos.com \
  thanosrulers.monitoring.coreos.com
```

## ⚙️ **D) Remove Leftovers**
```bash
kubectl delete ingress -n monitoring --all
kubectl delete secret -n monitoring --all
```

## 🧾 **E) Delete Namespace**
```bash
kubectl delete namespace monitoring
```

---

✅ **SOP Verified**  
This process deploys and manages the full Kube-Prometheus-Stack accessible at:
- Grafana → `grafanax.apsis.localnet`
- Prometheus → `prometheusx.apsis.localnet`
- Alertmanager → `alertmanagerx.apsis.localnet`
