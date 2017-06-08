# promql-examples
Example Queries for Prometheus

## CollectD

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
