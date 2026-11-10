# Configure Prometheus

This guide creates the Prometheus configuration file, runs Prometheus as a `systemd` service, and verifies that metrics are being collected. First complete [Install Prometheus and Node Exporter](./lab01-prometheus-installation.md) on the Prometheus host. To understand the YAML syntax and each configuration field, see [Understanding prometheus.yaml](../concepts/prometheus-yaml-explained.md).

## 1. Configure scrape endpoints

Create `/etc/prometheus/prometheus.yaml`:

```bash
sudo nano /etc/prometheus/prometheus.yaml
```

For a first run, copy the starter YAML from [Understanding prometheus.yaml](../concepts/prometheus-yaml-explained.md#starter-configuration) into this file and update the targets reachable from your Prometheus host. For a larger setup, use the [full multi-environment example](../concepts/prometheus-yaml-explained.md#full-example-vms-kubernetes-applications-and-databases) as a template, following its notes about exporter endpoints and Kubernetes access.

Set the configuration file to be readable by the service but not writable by it:

```bash
sudo chown root:prometheus /etc/prometheus/prometheus.yaml
sudo chmod 0640 /etc/prometheus/prometheus.yaml
```

## 2. Validate the configuration

Check the YAML and Prometheus configuration before starting the server:

```bash
sudo -u prometheus promtool check config /etc/prometheus/prometheus.yaml
```

Fix any reported errors before continuing.

## 3. Create the Prometheus systemd service

Create `/etc/systemd/system/prometheus.service`:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add:

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yaml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.listen-address=127.0.0.1:9090
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

The web interface is bound to localhost in this example. To access it remotely, use a properly secured reverse proxy or other access controls rather than exposing the Prometheus UI directly to an untrusted network.

## 4. Start Prometheus

Load the service, then start Prometheus now and automatically after a reboot:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

Check that it is running:

```bash
sudo systemctl status prometheus
```

If it fails to start, inspect the service logs:

```bash
sudo journalctl -u prometheus -n 50 --no-pager
```

## 5. Verify targets and query metrics

On the Prometheus host, open `http://127.0.0.1:9090/targets` in a browser. Prometheus and node_exporter should show as **UP** when their metrics endpoints are reachable. You can also check the server's readiness endpoint:

```bash
curl -f http://127.0.0.1:9090/-/ready
```

In the Prometheus expression browser at `http://127.0.0.1:9090/graph`, run:

```promql
up
```

The value is `1` when a target scrape succeeds and `0` when it fails. If a target is down, confirm the exporter or application is running, its `/metrics` endpoint is reachable from the Prometheus host, and the target address and port are correct.

## 6. Add or change endpoints

To monitor another endpoint, add a job under `scrape_configs` as described in [Understanding prometheus.yaml](../concepts/prometheus-yaml-explained.md#adding-an-application-endpoint). Use the actual hostname and port where your application exposes Prometheus-formatted metrics. After editing and saving the file, validate and restart Prometheus to load the change:

```bash
sudo -u prometheus promtool check config /etc/prometheus/prometheus.yaml
sudo systemctl restart prometheus
```

Then revisit `/targets` to confirm that the new target is **UP**. For dynamically changing environments such as Kubernetes, use Prometheus service discovery instead of maintaining a long static target list.
