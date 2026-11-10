# Connect Grafana to Prometheus

Grafana needs a data source to query Prometheus. Before starting, make sure Prometheus is running and accessible from the Grafana server. If you have not installed and configured Prometheus yet, follow [Install Prometheus and Node Exporter](./lab01-prometheus-installation.md) and [Configure Prometheus](./lab02-prometheus-configuration.md).

## 1. Find the Prometheus URL from Grafana's point of view

Use the URL that the **Grafana server** can reach:

| Setup | Prometheus URL in Grafana |
| :--- | :--- |
| Prometheus and Grafana run on the same host | `http://127.0.0.1:9090` |
| They run on separate hosts | `http://<prometheus-host-or-IP>:9090` |
| Prometheus runs in another container | Use its reachable service name and port, for example `http://prometheus:9090`, on a shared container network. |

`localhost` and `127.0.0.1` refer to the machine or network namespace where Grafana runs. They do not refer to the user's browser or another server. The URL must be reachable from the Grafana host.

The Prometheus installation guide binds Prometheus to `127.0.0.1:9090`, which supports Grafana on the same host. If Grafana runs on another host, configure Prometheus or a secured proxy to be reachable from that host, and restrict access to trusted clients. Do not expose Prometheus's unauthenticated endpoint directly to an untrusted network.

## 2. Add Prometheus as a data source

1. Sign in to Grafana.
2. Open **Connections** → **Data sources**.
3. Select **Add data source**.
4. Choose **Prometheus**.
5. Enter the Prometheus server URL in the **Prometheus server URL** field, such as `http://127.0.0.1:9090` when Grafana and Prometheus are on the same host.
6. Keep the default access mode unless your environment requires a different supported setup. Grafana's server-side access is appropriate for this guide.
7. Select **Save & test**.

Grafana should report that the data source is working. Menu names may differ slightly between Grafana versions.

## 3. Troubleshoot a failed connection

- Confirm Prometheus is running: `sudo systemctl status prometheus`.
- On the Grafana host, test the URL, for example: `curl -f http://127.0.0.1:9090/-/ready`.
- If the services are on different hosts, check DNS, routing, firewall rules, and the Prometheus listen address.
- If Grafana runs in a container, remember that `127.0.0.1` inside the container points to that container. Use a reachable service name or host address.
- Review Grafana logs with `sudo journalctl -u grafana-server -n 50 --no-pager`.

Once the data source test passes, use it when [creating a dashboard](./lab05-grafana-dashboard.md).
