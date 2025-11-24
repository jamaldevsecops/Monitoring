# Integrate Uptime‑Kuma with Prometheus & Grafana

> **Goal:** Scrape Uptime‑Kuma metrics with Prometheus and display them in Grafana using dashboard **ID 18278**.
> Source: https://github.com/louislam/uptime-kuma/wiki/Prometheus-Integration

---

## 🧰 Prerequisites

- Uptime‑Kuma running and reachable from the Prometheus server (example: `http://192.168.69.10:3000`).
- A valid Uptime‑Kuma API key (you created this and will use it as the password for Basic Auth in Prometheus).
- Prometheus installed and manageable (you can edit and reload `prometheus.yml`).
- Grafana with a Prometheus data source and the ability to import dashboards.

---

## ✅ Quick verification (before editing Prometheus)

1. From the Prometheus server, verify the metrics endpoint returns Prometheus metrics and is reachable:

```bash
# replace host:port with your Uptime-Kuma address
curl -sS -i http://192.168.69.10:3000/metrics | head -n 20
```

- Expected: a `200 OK` and text containing metric lines like `uptime_kuma_*` or similar.
- If you get `401` or `403`, authentication is required; Prometheus must send Basic Auth.

---

## 1) Prometheus job configuration

Add / update this job in your `prometheus.yml` under `scrape_configs`.

> **Note:** You already provided a job — below is a cleaned version with formatting and an optional `relabel_configs` block if you want to drop labels or rewrite the `instance` label.

```yaml
- job_name: "uptime"
  scrape_interval: 30s
  scheme: http
  metrics_path: "/metrics"
  static_configs:
    - targets: ["192.168.69.10:3000"]
  basic_auth:  # Only needed if authentication is enabled on Uptime-Kuma
    username: "admin@apsis.localnet"    # your Uptime-Kuma user
    password: "uk1_rnSE5AO0gYImcbFUsdfQmkYWSLcdQWxzyG"  # your API key (used as password)
  # optional: relabeling to set target label seen in Grafana/prometheus UI
  relabel_configs:
    - source_labels: [__address__]
      target_label: instance
```

**Important:** keep the `password` secret. If your Prometheus config lives in version control, use a secret mechanism (e.g., consul-template, vault, or Kubernetes secret) instead of plaintext.

After editing `prometheus.yml`, reload or restart Prometheus:

```bash
# if using systemd
sudo systemctl reload prometheus || sudo systemctl restart prometheus

# or use the HTTP reload endpoint
curl -X POST http://localhost:9090/-/reload

# if using docker
cd ~/prometheus && docker-compose restart
```

---

## 2) Confirm Prometheus is scraping the target

1. Open Prometheus UI: `http://<prometheus-host>:9090/targets`
2. Find the `uptime` job. You should see your `192.168.69.10:3000` target listed and `UP`.
3. If `DOWN`, click the `last scrape` error message to see the reason (network, 4xx/5xx, TLS, auth).

Troubleshooting common errors:

- `401 Unauthorized` / `403` → verify `basic_auth` username and password (API key) are correct.
- `dial tcp` → check network/firewall between Prometheus and Uptime‑Kuma.
- `tls` errors → if Uptime‑Kuma uses HTTPS, change `scheme` to `https` and configure TLS settings.

---

## 3) Add Prometheus as a data source in Grafana

1. Log in to Grafana.
2. Go to **Configuration → Data Sources → Add data source**.
3. Choose **Prometheus**.
4. Set **URL** to your Prometheus server, e.g. `http://<prometheus-host>:9090`.
5. Click **Save & test** — you should see a success message.

---

## 4) Import Grafana dashboard (ID **18278**)

1. In Grafana, go to **Create → Import**.
2. In **Import via grafana.com**, paste `18278` into the _Grafana.com dashboard_ field and click **Load**.
3. Select the Prometheus data source you added earlier from the **Prometheus** dropdown.
4. (Optional) Edit any dashboard variables (for example `instance`, `hostname`, or `job`) to match your `static_configs` target(s). If the dashboard expects `instance` variable, ensure your Prometheus target label `instance` matches your host:port or set `relabel_configs` to change it.
5. Click **Import**.

🎉 The dashboard should load and start showing Uptime‑Kuma metrics.

---

## 5) Map dashboard variables (if panels look empty)

- Many dashboards provide a top-left variable selector (e.g., `instance`).
- Set it to `192.168.69.10:3000` (or whatever label Prometheus shows under `instance`) to populate panels.
- If variable options are blank, open **Dashboard → Settings → Variables** and inspect the query used for the variable (change it to `label_values(instance)` or `label_values(job)` as needed).

---

## 6) Common extra tweaks and tips

- **Use HTTPS:** If Uptime‑Kuma is served via HTTPS, change `scheme: https` in Prometheus and optionally configure `tls_config`.

- **Bearer tokens / custom headers:** If Uptime‑Kuma expects the API key in a header rather than basic auth, Prometheus supports `authorization` header via `authorization:` in `http_sd_configs` or using `params`. But basic_auth is most common when Uptime‑Kuma uses its API key as password.

- **Relabeling for friendly names:** Use `relabel_configs` to set a friendlier `instance` label like `uptime-kuma-prod`.

```yaml
- source_labels: [__address__]
  regex: "192.168.69.10:3000"
  replacement: "uptime-kuma-1"
  target_label: instance
```

- **Security:** Use network rules or VPN so Prometheus can reach Uptime‑Kuma without exposing it publicly.

---

## 7) Troubleshooting checklist

- `curl http://<uptime-host>:<port>/metrics` returns valid metrics.
- Prometheus `targets` page shows the target and `UP`.
- Prometheus `graph` → run a sample metric like `up{job="uptime"}` should return a `1` time series for that target.
- Grafana data source → test connection shows OK.
- Grafana imported dashboard → variable `instance` is set to the scraped target.

---

## 8) Example useful Prometheus queries

- Uptime for the Uptime‑Kuma process (if exported):

```
up{job="uptime"}
```

- Any Uptime‑Kuma specific metric (replace `uptime_kuma_pings_total` with an actual metric name you found in `/metrics`):

```
rate(uptime_kuma_pings_total[5m])
```

---

## 🎯 Summary / Next steps

1. Verify `/metrics` on the Uptime‑Kuma host.
2. Add the Prometheus job (use the API key as `password` under `basic_auth`).
3. Reload Prometheus, confirm target is `UP`.
4. Add Prometheus as Grafana data source and import dashboard `18278`.
5. Set dashboard variables (e.g., `instance`) and enjoy the metrics.

If you want, I can:

- Provide a version of `prometheus.yml` with secrets templated for Kubernetes/Helm.
- Produce a Grafana provisioning JSON that auto-imports dashboard 18278 and wires the data source (for fully automated deployments).

---

*Created for you — happy monitoring!*

