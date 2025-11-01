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

## 🧠 Step 1: Create Values File

Save the following as **`values.yaml`**:

```yaml
# values.yaml for kube-prometheus-stack with Ingress and IP Whitelisting

# Global settings (optional; adjust as needed)
defaultRules:
  create: true
  rules:
    etcd: false  # Disable if not using etcd
    general: true
    kubeApiserver: false  # Enable if monitoring API server
    kubeScheduler: true
    kubelet: true
    kubernetesSystem: true
    kubeControllerManager: false

# Grafana configuration
grafana:
  enabled: true
  adminPassword: prom-operator  # Change this for production
  ingress:
    enabled: true
    ingressClassName: nginx  # Set to your Ingress class; omit if default
    annotations:
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
      # Add other annotations if needed, e.g., for auth or rewrite
    hosts:
      - grafana.apsis.localnet
    path: /
    pathType: Prefix
    tls: []  # Add TLS secrets if enabling HTTPS, e.g., - secretName: grafana-tls

# Prometheus configuration
prometheus:
  prometheusSpec:
    # Default scrape config; adjust as needed
    serviceMonitorSelectorNilUsesHelmValues: false
  ingress:
    enabled: true
    ingressClassName: nginx  # Set to your Ingress class; omit if default
    annotations:
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
      # Optional: Rewrite for subpaths if needed
      # nginx.ingress.kubernetes.io/rewrite-target: /
    hosts:
      - prometheus.apsis.localnet
    path: /
    pathType: Prefix
    tls: []  # Add TLS if needed

# Alertmanager configuration
alertmanager:
  enabled: true
  ingress:
    enabled: true
    ingressClassName: nginx  # Set to your Ingress class; omit if default
    annotations:
      nginx.ingress.kubernetes.io/whitelist-source-range: "192.168.0.0/24"
    hosts:
      - alertmanager.apsis.localnet
    path: /
    pathType: Prefix
    tls: []  # Add TLS if needed

# Dependencies (enabled by default)
kubeStateMetrics:
  enabled: true

nodeExporter:
  enabled: true

# Optional: Disable other components if not needed
# e.g., prometheusOperator:
#   enabled: true  # Keeps the operator
```

This configuration:
- Enables Grafana, Prometheus, and Alertmanager Ingress
- Restricts access to the `192.168.0.0/24` network only
- Uses NGINX ingress class
- No TLS (you can add cert-manager later)

---

## 🚀 Step 2: Install Kube Prometheus Stack

```bash
helm install monitoring oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack \
  -n monitoring \
  --create-namespace \
  -f values.yaml
```

This will deploy:
- Prometheus
- Alertmanager
- Grafana
- Node Exporters
- Kube-state metrics

---

## 🔍 Step 3: Verify Installation

```bash
kubectl get pods -n monitoring
```
Example Output: 
```
NAME                                                        READY   STATUS    RESTARTS   AGE
alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   0          13m
kube-prometheus-stack-grafana-64666c4bdd-g7hjv              3/3     Running   0          14m
kube-prometheus-stack-kube-state-metrics-557fd457c6-7v2kz   1/1     Running   0          14m
kube-prometheus-stack-operator-85c8966dbb-4t94r             1/1     Running   0          14m
kube-prometheus-stack-prometheus-node-exporter-mqwcx        1/1     Running   0          14m
kube-prometheus-stack-prometheus-node-exporter-tglz6        1/1     Running   0          14m
kube-prometheus-stack-prometheus-node-exporter-vmmr7        1/1     Running   0          14m
prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          13m
```
```bash
kubectl get svc -n monitoring
```
Example Output: 
```
NAME                                             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                            ClusterIP   None             <none>        9093/TCP,9094/TCP,9094/UDP   15m
kube-prometheus-stack-alertmanager               ClusterIP   10.108.125.105   <none>        9093/TCP,8080/TCP            15m
kube-prometheus-stack-grafana                    ClusterIP   10.96.195.71     <none>        80/TCP                       15m
kube-prometheus-stack-kube-state-metrics         ClusterIP   10.99.119.33     <none>        8080/TCP                     15m
kube-prometheus-stack-operator                   ClusterIP   10.104.76.210    <none>        443/TCP                      15m
kube-prometheus-stack-prometheus                 ClusterIP   10.106.123.0     <none>        9090/TCP,8080/TCP            15m
kube-prometheus-stack-prometheus-node-exporter   ClusterIP   10.111.173.13    <none>        9100/TCP                     15m
prometheus-operated                              ClusterIP   None             <none>        9090/TCP                     15m
```

```bash
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

## 🧑‍💻 Step 4: Access Dashboards

| Service | URL | Credentials |
|----------|-----|--------------|
| Grafana | http://grafana.apsis.localnet | admin / admin123 |
| Prometheus | http://prometheus.apsis.localnet | - |
| Alertmanager | http://alertmanager.apsis.localnet | - |

> ⚠️ Only clients in the `192.168.0.0/24` network will be allowed by the Ingress whitelist annotation.

---

## 🧹 Step 5: Uninstall

If you ever need to remove it:

```bash
helm uninstall kube-prometheus-stack -n monitoring
kubectl delete ns monitoring
```

---

✅ **Done!**  
You now have Grafana, Prometheus, and Alertmanager accessible with friendly hostnames and restricted network access.
