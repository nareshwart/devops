# PromQL Queries for the Prometheus UI

PromQL is Prometheus's query language. You can run these examples in Prometheus's **Graph** page, usually at `http://127.0.0.1:9090/graph`.

For a step-by-step explanation of metric selectors, label filters, range windows, functions, and aggregations, see [PromQL Query Basics](./promql-query-basics.md).

The examples assume your scrape jobs are named `prometheus` and `node`, as in the basic `prometheus.yaml`. If you changed a `job_name`, update the matching `job="..."` selector. Node-specific examples require node_exporter to be installed, scraped, and **UP**.

## Run a query

1. Open the Prometheus Graph page.
2. Enter a query in the expression field.
3. Select **Execute**.
4. Choose **Table** to inspect current values or **Graph** to see values over time.

Use the `/targets` page to check that the corresponding scrape target is **UP**. Prometheus only has data from metrics that are exposed by configured and successfully scraped targets.

## Start with target health

### Is each target up?

```promql
up
```

Returns `1` for a successful scrape and `0` for a failed scrape. The result includes `job` and `instance` labels that identify each target.

### Check node_exporter targets only

```promql
up{job="node"}
```

If your node exporter job is named `vm-nodes`, use `up{job="vm-nodes"}` instead.

### Count targets by job

```promql
count by (job) (up)
```

Counts the configured targets for each job. To count only healthy targets:

```promql
count by (job) (up == 1)
```

## Host CPU and memory

### CPU usage by host

```promql
100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
```

`node_cpu_seconds_total` is a counter of CPU time. `rate(...[5m])` estimates its per-second increase over five minutes; the query averages the idle fraction across CPU cores and converts non-idle time to a percentage.

### Memory used percentage by host

```promql
100 * (1 - (
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
))
```

This estimates memory in use using Linux's available-memory metric. The result is a percentage from 0 to 100.

## Disk and network

### Filesystem used percentage

```promql
100 * (
  1 -
  (
    node_filesystem_avail_bytes{
      fstype!~"tmpfs|overlay|squashfs",
      mountpoint!=""
    }
    /
    node_filesystem_size_bytes{
      fstype!~"tmpfs|overlay|squashfs",
      mountpoint!=""
    }
  )
)
```

Returns a value for each filesystem and host. The filters omit common temporary and container filesystems; adapt them to the filesystems you want to monitor.

### Network receive rate by host

```promql
sum by (instance) (
  rate(node_network_receive_bytes_total{device!="lo"}[5m])
)
```

Returns received bytes per second, excluding the loopback interface. In the Graph page, set the unit to **bytes/sec** when visualizing it.

### Network transmit rate by host

```promql
sum by (instance) (
  rate(node_network_transmit_bytes_total{device!="lo"}[5m])
)
```

## Prometheus and application metrics

### Prometheus HTTP request rate by handler

```promql
sum by (handler) (
  rate(prometheus_http_requests_total[5m])
)
```

Shows Prometheus's HTTP request rate per second, grouped by handler. This metric is available when the Prometheus server exposes it.

### Application request rate

If an application exports a counter named `http_requests_total`, query its rate like this:

```promql
sum by (job) (
  rate(http_requests_total[5m])
)
```

Metric names and labels depend on the application instrumentation. Replace `http_requests_total` and `job` with the names your application actually exposes.

## Optional exporter examples

These queries only work if the corresponding exporter or metrics component is installed, configured, and scraped. Metric names may differ by exporter version or configuration.

### PostgreSQL exporter health

```promql
pg_up
```

The PostgreSQL exporter commonly exposes `pg_up`, where `1` indicates it can connect to PostgreSQL.

### MySQL exporter health

```promql
mysql_up
```

The MySQL exporter commonly exposes `mysql_up`, where `1` indicates a successful database scrape.

## Kubernetes queries

Kubernetes queries require the metrics source named in each example to be installed and scraped by Prometheus. **Kubernetes service discovery by itself only finds scrape targets; it does not create Kubernetes object or container metrics.**

### Pods by namespace and phase

Requires **kube-state-metrics**:

```promql
count by (namespace, phase) (
  kube_pod_status_phase{phase=~"Pending|Running|Succeeded|Failed|Unknown"} == 1
)
```

Each result is a pod count for a namespace and phase.

### Pods that are not Ready

Requires **kube-state-metrics**:

```promql
kube_pod_status_ready{condition="true"} == 0
```

The output retains labels such as `namespace` and `pod` to identify pods that are not reporting Ready.

### Container restarts in the last hour

Requires **kube-state-metrics**:

```promql
sum by (namespace, pod) (
  increase(kube_pod_container_status_restarts_total[1h])
)
```

Returns the estimated number of restarts per pod over the last hour. Pods with no restarts may be omitted from the result.

### Deployments with unavailable replicas

Requires **kube-state-metrics**:

```promql
kube_deployment_spec_replicas
-
on (namespace, deployment)
kube_deployment_status_replicas_available
> 0
```

A positive value indicates replicas requested but not currently available. Results include namespace and deployment labels.

### CPU usage by pod and container

Requires **container runtime metrics** (commonly exposed by kubelet/cAdvisor and scraped by Prometheus):

```promql
sum by (namespace, pod, container) (
  rate(container_cpu_usage_seconds_total{
    container!="",
    container!="POD"
  }[5m])
)
```

The result is CPU usage in cores, averaged over five minutes, grouped by pod and container. The exact labels and availability depend on Kubernetes, runtime, and scrape configuration.

### Memory working set by pod

Requires **container runtime metrics** (commonly exposed by kubelet/cAdvisor):

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes{
    container!="",
    container!="POD"
  }
)
```

The result is bytes of working-set memory, grouped by namespace and pod. Set the display unit to bytes in a dashboard.

### Nodes reporting NotReady

Requires **kube-state-metrics**:

```promql
kube_node_status_condition{
  condition="Ready",
  status="false"
} == 1
```

The returned series identify nodes whose Ready condition is false.

### Pending pods by namespace

Requires **kube-state-metrics**:

```promql
sum by (namespace) (
  kube_pod_status_phase{phase="Pending"} == 1
)
```

### Containers in CrashLoopBackOff

Requires **kube-state-metrics**:

```promql
kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"} == 1
```

The result includes the namespace, pod, and container labels for matching containers.

### Unavailable replicas by deployment

Requires **kube-state-metrics**:

```promql
kube_deployment_status_replicas_unavailable > 0
```

This directly reports deployments with one or more unavailable replicas.

### DaemonSet pods unavailable

Requires **kube-state-metrics**:

```promql
kube_daemonset_status_number_unavailable
```

The value is the number of DaemonSet pods that are unavailable, labeled by namespace and DaemonSet.

### Difference between desired and current HPA replicas

Requires **kube-state-metrics**:

```promql
kube_horizontalpodautoscaler_status_desired_replicas
-
on (namespace, horizontalpodautoscaler)
kube_horizontalpodautoscaler_status_current_replicas
```

A positive value means the HPA currently desires more replicas than are running; a negative value means more replicas are running than currently desired.

### CPU usage by namespace

Requires **container runtime metrics** (commonly exposed by kubelet/cAdvisor):

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total{
    namespace!="",
    container!="",
    container!="POD"
  }[5m])
)
```

The result is CPU usage in cores summed across each namespace.

### Memory usage by namespace

Requires **container runtime metrics** (commonly exposed by kubelet/cAdvisor):

```promql
sum by (namespace) (
  container_memory_working_set_bytes{
    namespace!="",
    container!="",
    container!="POD"
  }
)
```

The result is working-set memory in bytes summed across each namespace.

### Persistent volume claim usage percentage

Requires kubelet volume statistics to be scraped:

```promql
100 *
  kubelet_volume_stats_used_bytes
  /
  kubelet_volume_stats_capacity_bytes
```

The result is percentage used for each persistent volume claim. If these metrics are missing, check that Prometheus can scrape the kubelet metrics endpoint and that the cluster exposes volume statistics.

If one of these queries returns no data, confirm its required metrics source is installed, scraped, and exposing the metric. Metric availability and labels can vary by component version and configuration.

## Useful PromQL building blocks

| Expression | Meaning |
| :--- | :--- |
| `metric_name` | Selects all series with that metric name. |
| `{job="node"}` | Filters a metric to a matching label value. |
| `{device!="lo"}` | Excludes series whose `device` label is `lo`. |
| `{mode=~"user|system"}` | Matches label values using a regular expression. |
| `[5m]` | Selects a five-minute range of samples; range selectors are used with functions such as `rate`. |
| `rate(counter[5m])` | Estimates the per-second increase of a counter over five minutes. Use `rate` for counters, not gauges. |
| `sum by (instance) (...)` | Adds values together while retaining one output series per `instance`. |
| `avg by (instance) (...)` | Averages values while retaining one output series per `instance`. |

## If a query returns no data

1. Check that the target is **UP** at `http://127.0.0.1:9090/targets`.
2. Run `up` and confirm the expected `job` and `instance` labels.
3. Search for the metric name in the Prometheus expression browser or check the target's `/metrics` output.
4. Confirm the right exporter or application is exposing that metric; not every job exposes node_exporter metrics.
5. Adjust the job and label selectors to match your configuration.
6. For range queries, select a time period that includes recent scrapes.

