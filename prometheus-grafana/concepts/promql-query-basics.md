# PromQL Query Basics

PromQL expressions are built by selecting metrics, filtering their labels, and optionally applying functions, aggregations, or arithmetic. Enter an expression in the Prometheus **Graph** page, usually at `http://127.0.0.1:9090/graph`.

For ready-to-run host, application, database, and Kubernetes examples, see [PromQL Queries for the Prometheus UI](./promql-queries.md).

## 1. Start with a metric name

A metric name selects a time series:

```promql
up
```

`up` is a built-in Prometheus metric. It has a value of `1` for a successful scrape and `0` for a failed scrape. A metric can have many time series, usually distinguished by labels such as `job` and `instance`.

To find available metric names, type a name or prefix in the Prometheus expression field and use its suggestions, or inspect a target's `/metrics` endpoint.

## 2. Filter with labels

Put label matchers in braces after the metric name:

```promql
up{job="node"}
```

This selects only `up` series whose `job` label is `node`.

| Matcher | Meaning | Example |
| :--- | :--- | :--- |
| `=` | Exact match | `{job="node"}` |
| `!=` | Does not equal | `{device!="lo"}` |
| `=~` | Matches a regular expression | `{namespace=~"prod|staging"}` |
| `!~` | Does not match a regular expression | `{fstype!~"tmpfs|overlay"}` |

Use the label names and values that actually appear on your series. For example, your scrape job name comes from `job_name` in `prometheus.yaml`.

## 3. Choose an instant or range of samples

An instant selector such as `up{job="node"}` returns the latest sample for each matching series.

Add a range selector to look back over recent samples:

```promql
http_requests_total{job="api"}[5m]
```

This selects five minutes of samples for each matching series. Range selectors are commonly passed to functions such as `rate` or `increase`; a bare range selector is not usually the final expression shown in a graph.

## 4. Apply functions to metrics

Counter metrics only increase (except when they reset, such as on a process restart). Use `rate` to estimate their per-second increase:

```promql
rate(http_requests_total{job="api"}[5m])
```

Use `increase` to estimate the total increase across a time window:

```promql
increase(http_requests_total{job="api"}[1h])
```

Gauges represent values that can rise or fall, such as memory in use. Query gauges directly rather than applying `rate`:

```promql
node_memory_MemAvailable_bytes
```

## 5. Aggregate series

Use `sum`, `avg`, `min`, or `max` to combine series. A `by (...)` clause controls which labels remain in the result:

```promql
sum by (job) (
  rate(http_requests_total[5m])
)
```

This calculates a request rate for each job by summing the matching series. Without `by (job)`, `sum(...)` combines all matching series into one value.

## 6. Combine results with arithmetic and comparisons

PromQL supports arithmetic operators such as `+`, `-`, `*`, and `/`:

```promql
100 * (1 - (
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
))
```

This calculates memory usage as a percentage. Comparisons can filter results:

```promql
up == 0
```

This returns only targets whose `up` value is zero.

When combining two vectors with different label sets, Prometheus matches series by their labels. Use vector-matching modifiers such as `on (namespace, deployment)` only when necessary and when those labels identify the corresponding series.

## Build a complete expression

The CPU usage query below combines the pieces step by step:

1. `node_cpu_seconds_total` selects CPU-time counter series from node_exporter.
2. `{mode="idle"}` keeps only idle CPU time.
3. `[5m]` selects five minutes of samples for each series.
4. `rate(...[5m])` converts each counter into an idle-time rate.
5. `avg by (instance)(...)` averages across CPU cores for each host.
6. `100 * (1 - ...)` converts the idle fraction into a non-idle percentage.

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

## Run and read a query in Prometheus

1. Open the **Graph** page.
2. Enter the expression and select **Execute**.
3. Use **Table** to inspect current values and their labels.
4. Use **Graph** to see how values change over the selected time range.
5. If the result is empty, check that the target is **UP**, the metric exists, and the label filters match the series.

The graph's selected time range controls how far back Prometheus evaluates the expression. A range selector such as `[5m]` separately controls the lookback window used by functions such as `rate`.

