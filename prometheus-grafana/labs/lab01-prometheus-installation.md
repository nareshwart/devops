# Install Prometheus and Node Exporter

This guide covers installing the Prometheus and node_exporter binaries on Ubuntu or Debian Linux. For configuring scrape endpoints, creating the Prometheus service, and verifying metrics, continue with [Configure Prometheus](./lab02-prometheus-configuration.md).

> **Before you begin:** These commands are for a Linux `amd64` host with `sudo`, `curl`, and `tar`. For another operating system or CPU architecture, choose the matching archive from the [Prometheus download page](https://prometheus.io/download/) and [node_exporter releases](https://github.com/prometheus/node_exporter/releases).

## 1. Download and install Prometheus

Choose a version from the [official Prometheus releases](https://github.com/prometheus/prometheus/releases). Replace `REPLACE_WITH_RELEASE_VERSION` with the version number, without a leading `v`.

```bash
PROM_VERSION="REPLACE_WITH_RELEASE_VERSION"
cd /tmp
curl -fLO "https://github.com/prometheus/prometheus/releases/download/v${PROM_VERSION}/prometheus-${PROM_VERSION}.linux-amd64.tar.gz"
tar -xzf "prometheus-${PROM_VERSION}.linux-amd64.tar.gz"
cd "prometheus-${PROM_VERSION}.linux-amd64"
```

Create a dedicated, unprivileged service account and the configuration and data directories:

```bash
sudo useradd --system --user-group --no-create-home --shell /usr/sbin/nologin prometheus
sudo install -d -o root -g prometheus -m 0750 /etc/prometheus
sudo install -d -o prometheus -g prometheus -m 0750 /var/lib/prometheus
```

If the `prometheus` user already exists, skip its `useradd` command.

Install the server and configuration-checking utility:

```bash
sudo install -o root -g root -m 0755 prometheus /usr/local/bin/prometheus
sudo install -o root -g root -m 0755 promtool /usr/local/bin/promtool
```

The Prometheus server will be configured and started in the [configuration guide](./lab02-prometheus-configuration.md).

## 2. Download and install node_exporter

node_exporter exposes Linux host metrics such as CPU, memory, filesystem, and network statistics for Prometheus to scrape. Choose a version from the [node_exporter releases](https://github.com/prometheus/node_exporter/releases). Replace `REPLACE_WITH_NODE_EXPORTER_VERSION` with the version number, without a leading `v`.

```bash
NODE_EXPORTER_VERSION="REPLACE_WITH_NODE_EXPORTER_VERSION"
cd /tmp
curl -fLO "https://github.com/prometheus/node_exporter/releases/download/v${NODE_EXPORTER_VERSION}/node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz"
tar -xzf "node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64.tar.gz"
cd "node_exporter-${NODE_EXPORTER_VERSION}.linux-amd64"
```

Create a dedicated service account and install the binary:

```bash
sudo useradd --system --user-group --no-create-home --shell /usr/sbin/nologin node_exporter
sudo install -o root -g root -m 0755 node_exporter /usr/local/bin/node_exporter
```

If the `node_exporter` user already exists, skip its `useradd` command.

## 3. Run node_exporter as a service

Create `/etc/systemd/system/node_exporter.service`:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add:

```ini
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --web.listen-address=127.0.0.1:9100
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

This binds node_exporter to localhost because the guide assumes Prometheus and node_exporter run on the same host. If they run on different hosts, configure node_exporter to listen on an address reachable from the Prometheus server and restrict port `9100` to that server using firewall or network security rules.

Load the service and start it automatically now and after reboots:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Check the service and its metrics endpoint:

```bash
sudo systemctl status node_exporter
curl -f http://127.0.0.1:9100/metrics
```

Once installed, continue with [Configure Prometheus](./lab02-prometheus-configuration.md) to create the YAML configuration, set up the Prometheus service, and confirm the targets are being scraped.
