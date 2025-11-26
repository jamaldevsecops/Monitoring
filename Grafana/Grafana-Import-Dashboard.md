




# 📊 Prometheus Queries in Megabytes per Second (MB/s)

Below are the converted queries for Disk Read, Disk Write, Network Receive, and Network Transmit — all in **MB/s**.

---

## 💽 Disk Read (MB/s)
```promql
max(
  rate(node_disk_read_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

---

## 💽 Disk Write (MB/s)
```promql
max(
  rate(node_disk_written_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

---

## 🌐 Network Receive (MB/s)
```promql
max(
  rate(node_network_receive_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

---

## 🌐 Network Transmit (MB/s)
```promql
max(
  rate(node_network_transmit_bytes_total{origin_prometheus=~"$origin_prometheus", job=~"$job"}[$interval])
  / (1024 * 1024)
) by (instance)
```

---

✔ All values are converted from **bytes/sec → MB/sec**  
