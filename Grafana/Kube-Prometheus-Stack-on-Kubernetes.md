# 📊 Kube Prometheus Stack Deployment with Helm
This guide installs **Kube Prometheus Stack** using Helm and configures **Ingress** resources to access Grafana and Prometheus dashboards using custom domain names (e.g., `grafana.apsis.localnet`, `prometheus.apsis.localnet`).

---

## 🧩 Prerequisites

- Kubernetes cluster (v1.25+ recommended)
- Helm 3.x installed
- Ingress Controller already configured (e.g., NGINX, Traefik)
- DNS or `/etc/hosts` entries for:
  ```
  grafana.apsis.localnet     → <ingress-controller-ip>
  prometheus.apsis.localnet  → <ingress-controller-ip>
  ```

---

## ⚙️ Step 1: Add Helm Repo and Update

```bash
helm repo list
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 📦 Step 2: Create Namespace

```bash
kubectl create namespace monitoring
```

---

## 🧠 Step 3: Create Values File

Save the following as **`values.yaml`**:

```yaml
grafana:
  enabled: true
  adminPassword: "admin123"
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
    hosts:
      - grafana.apsis.localnet
    path: /
    tls: []

prometheus:
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
    hosts:
      - prometheus.apsis.localnet
    path: /
    tls: []

alertmanager:
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
    hosts:
      - alertmanager.apsis.localnet
    path: /
    tls: []
```

This configuration:
- Enables Grafana, Prometheus, and Alertmanager Ingress
- Restricts access to the `192.168.0.0/24` network only
- Uses NGINX ingress class
- No TLS (you can add cert-manager later)

---

## 🚀 Step 4: Install Kube Prometheus Stack

```bash
helm search repo kube-prometheus-stack
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack   -n monitoring   -f values.yaml
```

This will deploy:
- Prometheus
- Alertmanager
- Grafana
- Node Exporters
- Kube-state metrics

---

## 🔍 Step 5: Verify Installation

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```

Example output:

```
NAME                                       READY   STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-stack-0       2/2     Running   0          2m
grafana-6f7f85c4c8-m7kvz                  1/1     Running   0          2m
prometheus-kube-prometheus-stack-prometheus-0   2/2   Running   0   2m

NAME                                     CLASS    HOSTS                      ADDRESS   PORTS   AGE
kube-prometheus-stack-grafana            nginx    grafana.apsis.localnet     10.0.0.5  80      2m
kube-prometheus-stack-prometheus         nginx    prometheus.apsis.localnet  10.0.0.5  80      2m
kube-prometheus-stack-alertmanager       nginx    alertmanager.apsis.localnet 10.0.0.5 80     2m
```

---

## 🧑‍💻 Step 6: Access Dashboards

| Service | URL | Credentials |
|----------|-----|--------------|
| Grafana | http://grafana.apsis.localnet | admin / admin123 |
| Prometheus | http://prometheus.apsis.localnet | - |
| Alertmanager | http://alertmanager.apsis.localnet | - |

> ⚠️ Only clients in the `192.168.0.0/24` network will be allowed by the Ingress whitelist annotation.

---

## 🧹 Step 7: Uninstall

If you ever need to remove it:

```bash
helm uninstall kube-prometheus-stack -n monitoring
kubectl delete ns monitoring
```

---

✅ **Done!**  
You now have Grafana, Prometheus, and Alertmanager accessible with friendly hostnames and restricted network access.
