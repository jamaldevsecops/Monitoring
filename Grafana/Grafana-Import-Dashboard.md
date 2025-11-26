
# 📘 SOP: Importing Grafana Dashboard (ID 11074) and Applying MB/s Queries with Thresholds

## 🟦 1. Import the Dashboard
1. Open **Grafana**.
2. Navigate to **Dashboards → New → Import**.
3. In **Import via grafana.com**, enter: `11074`.
4. Click **Load**.
5. Select your **Prometheus** data source.
6. Click **Import**.

---

## 🟧 2. Update Panels with MB/s Queries

Replace the default PromQL expressions with the following **Megabytes per Second (MB/s)** versions.

### 💽 Disk Read (MB/s)
```promql
max(
  rate(node_disk_read_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

### 💽 Disk Write (MB/s)
```promql
max(
  rate(node_disk_written_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

### 🌐 Network Receive (MB/s)
```promql
max(
  rate(node_network_receive_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

### 🌐 Network Transmit (MB/s)
```promql
max(
  rate(node_network_transmit_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

---

## 🟩 3. Configure Legend Names
Set the **Legend** fields to help with overrides:

- `Disk Read (MB/s) - {{instance}}`
- `Disk Write (MB/s) - {{instance}}`
- `Net Recv (MB/s) - {{instance}}`
- `Net Tx (MB/s) - {{instance}}`

---

## 🟪 4. Set Display Units
1. Open the panel.
2. Go to **Field → Unit**.
3. Select **Data rate → megabytes per second (MB/s)**.

---

## 🟥 5. Apply Thresholds via Field Overrides

### ✨ Steps
1. In the panel editor, scroll to **Overrides**.
2. Click **Add field override**.
3. Choose **Fields with name**.
4. Match the legend or use regex (example: `Disk Read (MB/s).*`).
5. Add property → **Thresholds**.

### 📊 Recommended Threshold Values

#### 💽 Disk Read / Write
| Color | Value (MB/s) |
|-------|--------------|
| 🟩 Green | 0 |
| 🟨 Yellow | 20 |
| 🟧 Orange | 100 |
| 🟥 Red | 200 |

#### 🌐 Network Receive / Transmit
| Color | Value (MB/s) |
|-------|--------------|
| 🟩 Green | 0 |
| 🟨 Yellow | 10 |
| 🟧 Orange | 50 |
| 🟥 Red | 200 |

---

## 🏁 6. Save & Validate
✔ Save your dashboard  
✔ Confirm threshold colors are applied  
✔ Use **Inspect → Data** to verify field names if an override doesn't apply  

---

## ✅ End of SOP
