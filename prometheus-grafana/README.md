# Prometheus & Grafana

Learn how Prometheus collects and stores metrics, and how Grafana queries and visualizes them. The concepts explain the architecture and configuration; the labs provide hands-on exercises.

## Concepts

| # | Topic | What you'll learn |
| :---: | :--- | :--- |
| 1 | [Prometheus Overview and Architecture](./concepts/prometheus-overview-arch.md) | Prometheus components, metric scraping, storage, queries, and alert flow |
| 2 | [Understanding `prometheus.yaml`](./concepts/prometheus-yaml-explained.md) | Configuration fields and examples for VMs, applications, databases, and Kubernetes |
| 3 | [What Is Grafana? Architecture Overview](./concepts/grafana-overview-arch.md) | How Grafana queries Prometheus and turns results into dashboards |

## Hands-on labs

| Lab | Topic | What you'll do |
| :---: | :--- | :--- |
| [Lab 01: Install Prometheus and Node Exporter](./labs/lab01-prometheus-installation.md) | Metrics collection setup | Install Prometheus and node_exporter and run node_exporter as a service |
| [Lab 02: Configure Prometheus](./labs/lab02-prometheus-configuration.md) | Scrape configuration | Configure `prometheus.yaml`, start Prometheus, and verify targets |
| [Lab 03: Install Grafana](./labs/lab03-grafana-installation.md) | Dashboard platform setup | Install Grafana and access its web interface |
| [Lab 04: Connect Grafana to Prometheus](./labs/lab04-grafana-prometheus-datasource.md) | Data source setup | Add Prometheus as a Grafana data source and test the connection |
| [Lab 05: Create a Grafana Dashboard](./labs/lab05-grafana-dashboard.md) | Metrics visualization | Build and save a host dashboard with CPU and memory panels |

## Suggested learning path

Read the concepts first, then complete Labs 01–05 in order. Labs 04–05 depend on Prometheus being installed and configured and Grafana being available.

## Repository folders

- [`concepts/`](./concepts/) — architecture and configuration explanations
- [`labs/`](./labs/) — step-by-step installation and configuration exercises
- [`img/`](./img/) — architecture diagrams used by the guides
