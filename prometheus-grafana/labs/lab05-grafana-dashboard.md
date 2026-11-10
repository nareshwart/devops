# Create a Grafana Dashboard

This guide creates a basic host-monitoring dashboard using metrics from Prometheus and node_exporter. First make sure the [Prometheus data source is configured](./lab04-grafana-prometheus-datasource.md) and the `node` target is **UP** on Prometheus's `/targets` page.

## 1. Create a dashboard

1. In Grafana, open **Dashboards**.
2. Select **New** → **New dashboard**.
3. Select **Add visualization**.
4. Choose the Prometheus data source you configured.

The exact button labels can vary slightly by Grafana version.

## 2. Add a CPU usage panel

1. In the query editor, select **Code** mode if Grafana opens the builder.
2. Enter this PromQL query:

   ```promql
   100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
   ```

3. Set the visualization to **Time series**.
4. Set the panel title to **CPU usage (%)** and the unit to **Percent (0-100)**.
5. Select **Run queries** or **Apply**, depending on the Grafana version.

The query calculates the percentage of CPU time not spent idle, averaged across CPU cores for each instance, over a rolling five-minute window.

## 3. Add a memory usage panel

1. Choose **Add** → **Visualization** from the dashboard.
2. Select the same Prometheus data source.
3. Enter:

   ```promql
   100 * (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes))
   ```

4. Choose **Time series** or **Gauge**, title it **Memory used (%)**, and set the unit to **Percent (0-100)**.
5. Apply the panel.

## 4. Save the dashboard

1. Select **Save dashboard**.
2. Give it a name, such as **Linux Host Overview**.
3. Choose a folder if prompted, then save.

Use the time picker to select the time range and the refresh control to choose how often Grafana refreshes the dashboard. These controls affect dashboard queries; Prometheus's scrape interval is configured separately in `prometheus.yaml`.

## 5. Check for missing data

- Confirm the `node` target is **UP** in Prometheus at `http://127.0.0.1:9090/targets`.
- In Grafana, confirm the Prometheus data source's **Save & test** check passes.
- Verify the metrics exist by querying `node_cpu_seconds_total` or `node_memory_MemTotal_bytes` in Prometheus.
- Check that the dashboard's selected time range includes recent samples.

For dashboards you need to share or deploy repeatedly, use Grafana's export or provisioning options rather than relying only on a manually created dashboard.
