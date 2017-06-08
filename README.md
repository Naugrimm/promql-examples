# promql-examples
Example Queries for [Prometheus](https://prometheus.io/)

## CollectD

These values are collected by using the [collectd-exporter](https://github.com/prometheus/collectd_exporter).

Install it with:

```
go get github.com/prometheus/collectd_exporter
```

Use the follwing systemd-file:

```systemd
[Unit]
Description=CollectD Exporter for Prometheus
After=network-online.target

[Service]
User=prometheus
Restart=on-failure
ExecStart=/usr/local/go-work/bin/collectd_exporter -collectd.listen-address=:25826

[Install]
WantedBy=multi-user.target
```

Enable start-on-boot:

```
systemctl daemon-reload
systemctl enable collectd-exporter
systemctl start collectd-exporter
```

### CPU

Returns the percentage of time at which all CPUs on the hosts were not idling.

```promql
100 * (1 - sum(collectd_cpu_total{type="idle"}) by (exported_instance) / sum(collectd_cpu_total) by (exported_instance))
```

The same query grouped by CPU cores for one host (replace `$server_name`):

```promql
100 * (1 - sum(collectd_cpu_total{exported_instance="$server_name", type="idle"}) by (cpu) / sum(collectd_cpu_total{exported_instance="$server_name"}) by (cpu))
```

Top 10 of non-idling CPUs on all servers:

```promql
topk(10, 100 * (1 - sum(collectd_cpu_total{type="idle"}) by (exported_instance, cpu) / sum(collectd_cpu_total) by (exported_instance, cpu)))
```

### Memory (RAM)

Returns the percentage of non-free memory on all servers:

```promql
100 * (1 - sum(collectd_memory{memory="free"}) by (exported_instance) / sum(collectd_memory) by (exported_instance))
```

If you want to count [cached](http://www.linuxatemyram.com/) memory as free use this:

```promql
100 * (1 - sum(collectd_memory{memory=~"free|cached"}) by (exported_instance) / sum(collectd_memory) by (exported_instance))
```

### Disk IOPS

```promql
sum(rate({__name__=~"collectd_disk_disk_ops_\\d_total",disk=~"vda\\d+"}[10m])) by (exported_instance, disk) > 0
```

### Disk Throughput

```promql
sum(rate({__name__=~"collectd_disk_disk_octets_\\d_total",disk=~"vda\\d+"}[10m])) by (exported_instance, disk) > 0
```

### Nginx

#### Active Connections

```promql
collectd_nginx_nginx_connections{nginx!="waiting"}
```

### Requests per second

```promql
rate(collectd_nginx_nginx_requests_total[2m])
```

### Apache

### Active Workers

```promql
collectd_apache_apache_scoreboard{type!="open"} > 0
```

### Requests per second

```promql
rate(collectd_apache_apache_requests_total[2m])
```
