## 19. Prometheus Installation on Ubuntu 22.04

### What is Prometheus?

Prometheus is an open-source monitoring and alerting tool used to collect, store, and query time-series metrics from servers, applications, and infrastructure.

It uses PromQL (Prometheus Query Language) to query metrics.

### Prerequisites

- Ubuntu 22.04 server
- Sudo/root access
- Internet connectivity
- Basic Linux commands knowledge

### Step 1: Update Ubuntu packages

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Create a Prometheus user

Create a dedicated system user to run Prometheus securely.

```bash
sudo groupadd --system prometheus

sudo useradd --system \
  --no-create-home \
  --shell /usr/sbin/nologin \
  --group prometheus prometheus
```

### Step 3: Create required directories

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

### Step 4: Download Prometheus

Check the latest stable release from:

https://prometheus.io/download/

Set the release version you selected:

```bash
PROM_VERSION=<VERSION>
```

Download and extract the Linux AMD64 package:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v${PROM_VERSION}/prometheus-${PROM_VERSION}.linux-amd64.tar.gz

tar -xvf prometheus-${PROM_VERSION}.linux-amd64.tar.gz

cd prometheus-${PROM_VERSION}.linux-amd64
```

Replace `<VERSION>` with the actual release version, without the `v` prefix.

### Step 5: Move Prometheus binaries

```bash
sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
```

Set ownership:

```bash
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

### Step 6: Move configuration files

```bash
sudo mv consoles /etc/prometheus/
sudo mv console_libraries /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/
```

Set permissions and ownership:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

### Step 7: Configure Prometheus

Open the configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Example configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

**Explanation:**

- `global`: Defines global monitoring settings.
- `scrape_interval`: Prometheus collects metrics every 15 seconds.
- `scrape_configs`: Defines which targets Prometheus monitors.
- `job_name`: Identifies the monitoring job.
- `targets`: Specifies the server and port from which metrics are collected.

### Step 8: Create a Prometheus systemd service

Create the service file:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add the following configuration:

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
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Save and exit.

### Step 9: Reload systemd and start Prometheus

```bash
sudo systemctl daemon-reload

sudo systemctl enable prometheus

sudo systemctl start prometheus
```

### Step 10: Verify Prometheus service

```bash
sudo systemctl status prometheus
```

Check whether Prometheus is listening on port 9090:

```bash
sudo ss -tulnp | grep 9090
```

Check the logs:

```bash
sudo journalctl -u prometheus -f
```

### Step 11: Access Prometheus Web UI

Prometheus uses port `9090` by default.

Open the browser:

```text
http://<SERVER-IP>:9090
```

For local access:

```text
http://localhost:9090
```

If accessing remotely, allow port 9090 only from trusted IP addresses in the server firewall or AWS Security Group. Avoid exposing the Prometheus UI publicly without appropriate access controls.

### Step 12: Verify Prometheus targets

1. Open the Prometheus web interface.
2. Navigate to **Status → Targets**.
3. Check whether the Prometheus target is UP.
4. Open the query page.
5. Execute:

```promql
up
```

If the target is healthy, Prometheus returns a value of `1`.

### Useful Prometheus Commands

```bash
# Start Prometheus
sudo systemctl start prometheus

# Stop Prometheus
sudo systemctl stop prometheus

# Restart Prometheus
sudo systemctl restart prometheus

# Check service status
sudo systemctl status prometheus

# Enable service at boot
sudo systemctl enable prometheus

# View logs
sudo journalctl -u prometheus -f

# Validate configuration
promtool check config /etc/prometheus/prometheus.yml
```

### Important Interview Questions

**Q1. What is Prometheus?**

Prometheus is an open-source monitoring and alerting tool that collects and stores time-series metrics from infrastructure and applications.

**Q2. What is PromQL?**

PromQL is the query language used to retrieve and analyze metrics stored in Prometheus.

**Q3. What is a scrape interval?**

It is the frequency at which Prometheus collects metrics from configured targets.

**Q4. What is a Prometheus target?**

A target is an endpoint from which Prometheus collects metrics, such as an application server or Node Exporter endpoint.

**Q5. What is Node Exporter?**

Node Exporter is a Prometheus exporter that exposes Linux system metrics such as CPU, memory, disk, and network usage.

**Q6. What is the default Prometheus port?**

Prometheus uses port `9090` by default.

**Q7. What is the difference between Prometheus and Grafana?**

Prometheus collects and stores metrics, while Grafana visualizes metrics through dashboards and graphs.

### Monitoring Workflow

```text
Linux Server
     |
     v
Node Exporter
     |
     v
Prometheus
     |
     v
Grafana
     |
     v
Monitoring Dashboard
```

**Reference:** https://www.cherryservers.com/blog/install-prometheus-ubuntu

**Official documentation:** https://prometheus.io/docs/prometheus/latest/getting_started/

**Note:** These are installation and learning instructions. Validate the release version, permissions, configuration, and firewall rules in your environment before using them in production.
