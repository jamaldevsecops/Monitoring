# 🚀 Deploying kube-prometheus-stack (with External Ingress)

## 🧩 1. Prerequisites
Make sure you have:
- A working **Kubernetes cluster**  
- **Helm 3.8+** installed  
- **NGINX Ingress Controller** running  
- A namespace for monitoring:
  ```bash
  kubectl create namespace monitoring
  ```

---

## ⚙️ 2. Create `values.yaml`

Since you’re using your own external `Ingress` objects, **disable** the built-in ingress in the Helm chart.

```yaml
# values.yaml
grafana:
  enabled: true
  adminPassword: "admin123"
  ingress:
    enabled: false

prometheus:
  ingress:
    enabled: false

alertmanager:
  ingress:
    enabled: false
```

---

## 🧭 3. Install the Chart

```bash
helm install monitoring oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack   -n monitoring   -f values.yaml
```

✅ This deploys kube-prometheus-stack into the `monitoring` namespace.

---

## 🌐 4. Create External Ingress Resources

You’ll manually expose Grafana, Prometheus, and Alertmanager through your existing NGINX Ingress controller.

### Grafana
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-external
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: grafana.apsis.localnet
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-grafana
            port:
              number: 80
```

### Prometheus
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-external
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: prometheus.apsis.localnet
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-kube-prometheus-prometheus
            port:
              number: 9090
```

### Alertmanager
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alertmanager-external
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: alertmanager.apsis.localnet
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-kube-prometheus-alertmanager
            port:
              number: 9093
```

Apply them:
```bash
kubectl apply -f grafana-ingress.yaml
kubectl apply -f prometheus-ingress.yaml
kubectl apply -f alertmanager-ingress.yaml
```

---

## 🔍 5. Verify Access

```bash
kubectl get ingress -n monitoring
```

Add your ingress IP to `/etc/hosts` if needed:

```
<INGRESS-IP> grafana.apsis.localnet
<INGRESS-IP> prometheus.apsis.localnet
<INGRESS-IP> alertmanager.apsis.localnet
```

Access via browser:
- Grafana → http://grafana.apsis.localnet  
- Prometheus → http://prometheus.apsis.localnet  
- Alertmanager → http://alertmanager.apsis.localnet  

---

# 🧹 Uninstalling kube-prometheus-stack

When you’re done or want to redeploy cleanly, follow these steps:

### 1️⃣ Uninstall Helm Release
Check the installed release:
```bash
helm list -n monitoring
```

Then uninstall it (replace `monitoring` if your release name differs):
```bash
helm uninstall monitoring -n monitoring
```

---

### 2️⃣ Delete CRDs (optional — for full cleanup)
```bash
kubectl delete crd   alertmanagers.monitoring.coreos.com   podmonitors.monitoring.coreos.com   probes.monitoring.coreos.com   prometheuses.monitoring.coreos.com   prometheusrules.monitoring.coreos.com   servicemonitors.monitoring.coreos.com   thanosrulers.monitoring.coreos.com
```

> ⚠️ Only do this if you’re sure no other apps use these CRDs.

---

### 3️⃣ Delete Namespace (optional)
```bash
kubectl delete namespace monitoring
```

---

### ✅ Summary

| Action | Command |
|--------|----------|
| Install | `helm install monitoring oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack -n monitoring -f values.yaml` |
| Create Ingress | `kubectl apply -f grafana-ingress.yaml -f prometheus-ingress.yaml -f alertmanager-ingress.yaml` |
| Uninstall | `helm uninstall monitoring -n monitoring` |
| Delete CRDs | `kubectl delete crd ...` |
| Delete Namespace | `kubectl delete namespace monitoring` |
