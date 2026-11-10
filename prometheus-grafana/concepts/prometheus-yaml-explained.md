# Understanding `prometheus.yaml`

Prometheus reads its settings from a YAML configuration file. This guide explains every field in the starter configuration, how the fields fit together, and how to add a scrape endpoint. The configuration filename is often `prometheus.yml`; this documentation uses `/etc/prometheus/prometheus.yaml`. Either extension works as long as the path passed to Prometheus matches the file.

## Starter configuration

Use this as a starting point for a Prometheus server and node_exporter running on the same host:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["127.0.0.1:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["127.0.0.1:9100"]
```

## What each field means

| Field | Where it appears | What it controls |
| :--- | :--- | :--- |
| `global` | Top level | Settings that apply by default across the Prometheus server. Job-specific settings can override applicable global defaults. |
| `scrape_interval` | Under `global` | How often Prometheus collects metrics from each target. `15s` means once every 15 seconds. A shorter interval gives more frequent samples but increases collection and storage load. |
| `evaluation_interval` | Under `global` | How often Prometheus evaluates configured recording and alerting rules. It does not control how often metrics are scraped. |
| `scrape_configs` | Top level | A list of scrape jobs: groups of targets that Prometheus collects using the same job settings. |
| `job_name` | In a scrape job | A name for the job. Prometheus adds it to each collected sample as the `job` label. Use a distinct, descriptive name for each job. |
| `static_configs` | In a scrape job | A list of targets entered manually. It is suitable for a small, fixed set of endpoints; dynamic environments can use service discovery instead. |
| `targets` | In a static configuration | Hostnames or IP addresses and ports where metrics endpoints are listening. Prometheus requests each target's metrics endpoint. |
| `labels` | In a static configuration | Optional key-value labels added to metrics from those targets, such as `environment: "development"`. |
| `metrics_path` | In a scrape job | Optional HTTP path for metrics. It defaults to `/metrics`; set it when an application exposes metrics at a different path. |

The starter file contains two jobs:

- The `prometheus` job scrapes the Prometheus server itself on port `9090`.
- The `node` job scrapes node_exporter on port `9100`.

The addresses use `127.0.0.1`, which means both services are on the same machine as Prometheus. For a service on another host, use a reachable address such as `192.168.1.20:9100`. The target must be accessible over the network from the Prometheus server.

## Full example: VMs, Kubernetes, applications, and databases

The following template combines several common target types in one file. Replace the example hostnames, ports, and Kubernetes access settings with values from your environment before using it.

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s

scrape_configs:
  # Prometheus monitors its own metrics.
  - job_name: "prometheus"
    static_configs:
      - targets: ["127.0.0.1:9090"]
        labels:
          environment: "production"

  # node_exporter runs on each VM and exposes host-level metrics.
  - job_name: "vm-nodes"
    static_configs:
      - targets:
          - "vm-app-01.example.net:9100"
          - "vm-app-02.example.net:9100"
        labels:
          environment: "production"
          platform: "virtual-machine"

  # The application exposes Prometheus metrics directly.
  - job_name: "application"
    metrics_path: "/metrics"
    static_configs:
      - targets: ["app-01.example.net:8080"]
        labels:
          environment: "production"

  # Database exporters expose database metrics; Prometheus does not scrape
  # the database protocol or database port directly.
  - job_name: "postgres-exporter"
    static_configs:
      - targets: ["db-01.example.net:9187"]
        labels:
          environment: "production"
          database: "postgres"

  - job_name: "mysql-exporter"
    static_configs:
      - targets: ["db-02.example.net:9104"]
        labels:
          environment: "production"
          database: "mysql"

  # Kubernetes pod discovery. This example assumes Prometheus runs outside
  # the cluster and can read the credentials in this kubeconfig.
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
        kubeconfig_file: "/etc/prometheus/kubeconfig"
    relabel_configs:
      # Only scrape pods that explicitly opt in.
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: "true"

      # Require the pod's Prometheus port annotation.
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: keep
        regex: "[0-9]+"

      # Use the annotated port instead of the pod's discovered container port.
      - source_labels:
          - __address__
          - __meta_kubernetes_pod_annotation_prometheus_io_port
        action: replace
        regex: '([^:]+)(:[0-9]+)?;([0-9]+)'
        replacement: "$1:$3"
        target_label: __address__

      # Use an optional custom path; otherwise the default is /metrics.
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        regex: "(.+)"
        target_label: __metrics_path__

      # Keep useful Kubernetes identity labels on scraped series.
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod
```

### What the additional fields do

| Field | What it controls |
| :--- | :--- |
| `scrape_timeout` | Maximum time Prometheus waits for one scrape. It must not be longer than the scrape interval. |
| `kubernetes_sd_configs` | Discovers targets from a Kubernetes API instead of requiring a fixed list of pod IP addresses. |
| `role: pod` | Discovers Kubernetes pods as scrape targets. |
| `kubeconfig_file` | Path to Kubernetes credentials and cluster connection settings. The Prometheus service account must be able to read this file. |
| `relabel_configs` | Keeps, changes, or drops discovered targets and labels before Prometheus scrapes them. |
| `source_labels` | Labels whose values are read as input to a relabeling step. |
| `action: keep` | Keeps a discovered target only when its source-label values match `regex`. |
| `regex` | Pattern used by a relabeling step to decide whether to keep or transform a target. |
| `replacement` | Value written to the label named by `target_label`; `$1`, `$2`, etc. refer to captured parts of the regex match. |
| `target_label` | Label to write, such as the special `__address__` target address or a useful `namespace` label. |

### Requirements and adjustments

- The VM targets must run node_exporter and be reachable from Prometheus on port `9100`.
- Install the appropriate database exporter near each database, configure it to collect from the database, and expose its metrics endpoint. Common defaults are `9187` for `postgres_exporter` and `9104` for `mysqld_exporter`; use the port configured in your environment.
- An application can be scraped directly only if it exposes Prometheus-formatted metrics. Otherwise, use an appropriate exporter.
- For Kubernetes, pods must include annotations such as `prometheus.io/scrape: "true"` and `prometheus.io/port: "8080"`. The optional `prometheus.io/path` annotation changes the metrics path.
- The Prometheus process needs Kubernetes API access with permission to list and watch pods. Use a least-privilege identity. Protect the kubeconfig and make it readable by the Prometheus service account.
- If Prometheus runs inside Kubernetes with a suitably permissioned service account, remove `kubeconfig_file` to use in-cluster credentials. If it runs outside the cluster, provide a valid kubeconfig and ensure the cluster API is reachable.
- The `127.0.0.1:9090` self-scrape target assumes Prometheus runs directly on the same host. If Prometheus runs as a pod, use an address reachable from that pod or remove the self-scrape job.
- The sample uses illustrative `.example.net` hostnames. Replace them with real addresses; targets that do not exist will appear **DOWN**.

## Adding an application endpoint

Add another item under `scrape_configs`. For example, if an application exposes Prometheus metrics on port `8080` at the default `/metrics` path:

```yaml
  - job_name: "my-application"
    static_configs:
      - targets: ["app-server.example.test:8080"]
        labels:
          environment: "development"
```

If the application exposes metrics at a path other than `/metrics`, set `metrics_path` in that job:

```yaml
  - job_name: "my-application"
    metrics_path: "/custom/metrics"
    static_configs:
      - targets: ["app-server.example.test:8080"]
```

Keep indentation consistent: YAML uses spaces to show which fields belong under `global`, a job, or a target group. Do not use tabs.

## How to edit and apply changes

1. Edit the configuration file, for example:

   ```bash
   sudo nano /etc/prometheus/prometheus.yaml
   ```

2. Check the configuration before applying it:

   ```bash
   sudo -u prometheus promtool check config /etc/prometheus/prometheus.yaml
   ```

3. If the check succeeds, restart Prometheus to load the updated configuration:

   ```bash
   sudo systemctl restart prometheus
   ```

4. Open `http://127.0.0.1:9090/targets` and confirm each expected target is **UP**.

If a target is **DOWN**, check that the service is running, the target address and port are correct, the metrics path is correct, and the Prometheus host can reach the endpoint. For more information about scrape configuration options, see the [Prometheus configuration reference](https://prometheus.io/docs/prometheus/latest/configuration/configuration/).
