# Install Grafana

This guide installs Grafana OSS on Ubuntu or Debian Linux using Grafana's official APT repository.

> **Before you begin:** You need `sudo` access and a supported 64-bit Linux system with network access. For other operating systems and current platform-specific instructions, see the [official Grafana installation documentation](https://grafana.com/docs/grafana/latest/setup-grafana/installation/).

## 1. Add Grafana's package signing key

Install the tools needed to download and install Grafana's repository signing key:

```bash
sudo apt-get update
sudo apt-get install -y curl gnupg
```

Create the keyring directory, download the official signing key, and install it for APT to verify packages from the Grafana repository:

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://apt.grafana.com/gpg.key -o /tmp/grafana.gpg.key
sudo gpg --dearmor --yes -o /etc/apt/keyrings/grafana.gpg /tmp/grafana.gpg.key
sudo chmod 0644 /etc/apt/keyrings/grafana.gpg
rm /tmp/grafana.gpg.key
```

## 2. Add the stable Grafana repository

```bash
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" \
  | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
```

## 3. Install Grafana

```bash
sudo apt-get install -y grafana
```

## 4. Start Grafana and enable it at boot

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now grafana-server
```

Check that the service is running:

```bash
sudo systemctl status grafana-server
```

If it does not start, inspect its logs:

```bash
sudo journalctl -u grafana-server -n 50 --no-pager
```

## 5. Open Grafana

From a browser that can reach the Grafana host, visit:

```text
http://<grafana-host>:3000
```

Grafana listens on port `3000` by default. Sign in with the initial account `admin` / `admin`, then set a new password when prompted. Do not retain default credentials.

Restrict access to port `3000` to trusted users or networks using your firewall and network controls. For production, use HTTPS and an appropriate authentication method; do not expose an unauthenticated Grafana service to the public internet.

Next, [connect Grafana to Prometheus](./lab04-grafana-prometheus-datasource.md), then [create a dashboard](./lab05-grafana-dashboard.md).
