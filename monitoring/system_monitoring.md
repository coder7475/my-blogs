

# How to set up a system monitoring for linux server using node exporter, prometheus & grafana cloud dashboard

---
To set up system monitoring for a Linux VPS using Node Exporter, Prometheus, and Grafana Cloud Dashboard, follow these steps:

## **1. Install and Configure Node Exporter**

Node Exporter collects system metrics from the Linux VPS.

1. **Download Node Exporter**:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.0/node_exporter-1.9.0.linux-amd64.tar.gz
```

2. **Extract and Move the Binary**:

```bash
tar -xvf node_exporter-1.9.0.linux-amd64.tar.gz
sudo mv node_exporter-1.9.0.linux-amd64/node_exporter /usr/local/bin/
```

3. **Create a System User**:

```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
```

4. **Set Up Node Exporter as a Service**:
Create the service file `/etc/systemd/system/node_exporter.service`:

```bash
sudo vim /etc/systemd/system/node_exporter.service
```

Add the following content:

```ini
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
After=network.target
Wants=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple

ExecStart=/usr/local/bin/node_exporter

# Security Setting
ProtectHome=true 
NoNewPrivileges=true
ProtectSystem=strict 
PrivateTmp=true 
PrivateDevices=true 
ProtectKernelTunables=true 
ProtectKernelModules=true 
ProtectControlGroups=true 
RestrictNamespaces=true 
RestrictRealtime=true 
RestrictSUIDSGID=true 

# Resource limits 
CPUQuota=10% 
MemoryLimit=50M 
LimitNOFILE=4096
LimitNPROC=4096
LimitCORE=100M 

# Restart configuration 
Restart=always 
RestartSec=5

[Install]
WantedBy=multi-user.target
```

5. **Start and Enable Node Exporter**:

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
sudo systemctl status node_exporter
```

6. **Verify Node Exporter**:
Ensure it is running on port `9100`:

```bash
ss -aplnt | grep 9100
```
Ensure it is exporting metrices:
```bash
curl http://localhost:9100/metrics
```

---

## **2. Install and Configure Prometheus**

Prometheus scrapes metrics from Node Exporter.

1. **Download Prometheus**:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.2.0/prometheus-3.2.0.linux-amd64.tar.gz
```


2. **Extract and Move Files**:

```bash
# Extract 
tar -xvf prometheus-3.2.0.linux-amd64.tar.gz

# Move Prometheus Binaries to System Path
sudo mv prometheus-3.2.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-3.2.0.linux-amd64/promtool /usr/local/bin/

# Create Configuration and Data Directories
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo mkdir -p /var/lib/prometheus/data
sudo mkdir -p /var/lib/prometheus/query_log

# Move Configuration File
sudo mv prometheus-3.2.0.linux-amd64/prometheus.yml /etc/prometheus/
```

3. **Edit the Configuration File** (`/etc/prometheus/prometheus.yml`):
Run the command below to open the file:
```bash
sudo vim /etc/prometheus/prometheus.yml
```
Add the Node Exporter's target:

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
    scrape_interval: 5s
    scrape_timeout: 5s
```

4. **Set Up Prometheus as a Service**:
Create a system user:
```bash
sudo useradd -rs /bin/false prometheus
```

Set required file permission:
```bash
sudo chown prometheus:prometheus /etc/prometheus 
sudo chown prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chmod -R 755 /var/lib/prometheus
```

Create `/etc/systemd/system/prometheus.service`:

```bash
sudo vim /etc/systemd/system/prometheus.service
```

Add this content:

```ini
[Unit]
Description=Prometheus Monitoring System
Documentation=https://prometheus.io/docs/prometheus/latest/installation/
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
WorkingDirectory=/var/lib/prometheus

ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/data \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --query.lookback-delta=5m \
    --enable-feature=memory-snapshot-on-shutdown

# Security Settings
ProtectHome=true
NoNewPrivileges=true
ProtectSystem=false
PrivateTmp=true
PrivateDevices=true
ProtectKernelTunables=true
ProtectKernelModules=true
ProtectControlGroups=true
RestrictNamespaces=true
RestrictRealtime=true
RestrictSUIDSGID=true

# Resource Limits
CPUQuota=10%
MemoryLimit=500M
LimitNOFILE=65535
LimitNPROC=4096
LimitCORE=infinity

# Restart Configuration
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

5. **Start and Enable Prometheus**:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus
```

6. **Verify Prometheus**:

Run the command below to verify its running :
```bash
ss -aplnt | grep 9090
curl localhost:9090/metrics
```


---

## **3. Set Up Grafana Cloud Dashboard**

Grafana visualizes metrics collected by Prometheus.

1. **Sign Up for Grafana Cloud**:
    - Create an account on Grafana Cloud (if you don’t already have one).
    - Obtain your organization’s unique *Prometheus remote write URL* and API key. 
2. **Configure Prometheus for Remote Write**:
Edit `/etc/prometheus/prometheus.yml` to include the remote write configuration:

```yaml
remote_write:
  - url: "https://prometheus-blocks-prod-us-central1.grafana.net/api/v1/write"
    basic_auth:
      username: "<your-grafana-cloud-instance-id>"
      password: "<your-api-key>"
```

3. **Restart Prometheus**:
Apply the changes by restarting Prometheus:

```bash
sudo systemctl restart prometheus
```

4. **Import Dashboards in Grafana Cloud**:
    - Log in to your Grafana Cloud instance.
    - Add Prometheus as a data source using your *remote write URL* if its not available in data source.
    - Import pre-built dashboards for Node Exporter (https://grafana.com/grafana/dashboards/1860-node-exporter-full/).

---

## **4. Verify Setup**

- Access Grafana Cloud at `https://<your-grafana-instance>.grafana.net`.
- View metrics such as CPU usage, memory consumption, disk I/O, etc., in real-time on your imported dashboards.

This setup provides comprehensive monitoring for your Linux VPS with minimal manual intervention[^1][^2][^4].

#### References

[^1]: https://www.liquidweb.com/blog/install-prometheus-node-exporter-on-linux-almalinux/

[^2]: https://www.idevopz.com/step-by-step-guide-to-install-prometheus-nodeexporter-grafana-on-a-ubuntu-20-04/

[^3]: https://github.com/narethim/tools/blob/master/setup-prometheus-node-exporter-arm64.md

[^4]: https://betterstack.com/community/guides/monitoring/visualize-prometheus-metrics-grafana/

[^5]: https://grafana.com/docs/grafana/latest/setup-grafana/set-up-grafana-monitoring/

[^6]: https://grafana.com/docs/grafana/latest/getting-started/get-started-grafana-prometheus/

[^7]: https://www.digitalocean.com/community/tutorials/how-to-add-a-prometheus-dashboard-to-grafana

[^8]: https://dev.to/kaitoii11/remote-writing-your-prometheus-metrics-to-grafana-cloud-3k24

[^9]: https://prometheus.io/docs/visualization/grafana/

[^10]: https://grafana.com/docs/grafana-cloud/connect-externally-hosted/data-sources/prometheus/configure-prometheus-data-source/

[^11]: https://grafana.com/docs/grafana-cloud/send-data/metrics/metrics-prometheus/prometheus-config-examples/noagent_linuxnode/

[^12]: https://www.howtoforge.com/how-to-install-prometheus-and-node-exporter-on-debian-12/

[^13]: https://prometheus.io/docs/guides/node-exporter/

[^14]: https://sbcode.net/prometheus/prometheus-node-exporter/

[^15]: https://www.stackhero.io/en/services/Prometheus/documentations/Using-Node-Exporter/Install-Prometheus-Node-Exporter-on-a-Linux-server

[^16]: https://www.youtube.com/watch?v=z8DKWHENn4s

[^17]: https://shape.host/resources/install-prometheus-and-node-exporter-on-debian-12-a-comprehensive-guide

[^18]: https://dev.to/mutuwa99/monitoring-linux-servers-with-node-exporter-prometheus-and-grafana-a-comprehensive-guide-1p35

[^19]: https://github.com/prometheus/node_exporter

[^20]: https://www.stackhero.io/en/services/Prometheus/documentations/Using-Node-Exporter

[^21]: https://betterstack.com/community/guides/monitoring/monitor-linux-prometheus-node-exporter

[^22]: https://grafana.com/docs/grafana-cloud/connect-externally-hosted/data-sources/prometheus/

[^23]: https://grafana.com/docs/k6/latest/results-output/real-time/grafana-cloud-prometheus/

[^24]: https://grafana.com/go/grafana-cloud-prometheus-1/

[^25]: https://grafana.com/static/img/docs/getting-started/simple_grafana_prom_dashboard.png?sa=X\&ved=2ahUKEwjwifbOl9uLAxUvm68BHVkeCswQ_B16BAgFEAI

[^26]: https://www.youtube.com/watch?v=_MbB8IVKMfw

[^27]: https://community.grafana.com/t/how-to-push-prometheus-metrics-in-grafana-cloud/47297

[^28]: https://www.youtube.com/watch?v=EGgtJUjky8w

[^29]: https://www.tigera.io/learn/guides/prometheus-monitoring/prometheus-grafana/

[^30]: https://grafana.com/docs/grafana-cloud/send-data/metrics/metrics-prometheus/

[^31]: https://www.youtube.com/watch?v=sDmf5YpyGms

[^32]: https://grafana.com/docs/learning-journeys/prometheus/

[^33]: https://grafana.com/static/img/docs/getting-started/simple_grafana_prom_dashboard.png?sa=X\&ved=2ahUKEwj77fbOl9uLAxXJqFYBHd_DH9gQ_B16BAgDEAI

[^34]: https://gist.github.com/nwesterhausen/d06a772cbf2a741332e37b5b19edb192

[^35]: https://frappedevops.hashnode.dev/install-and-configure-node-exporter-on-the-main-server

[^36]: https://www.fosstechnix.com/install-prometheus-node-exporter-on-linux

[^37]: https://ourcodeworld.com/articles/read/1686/how-to-install-prometheus-node-exporter-on-ubuntu-2004

[^38]: https://developer.couchbase.com/tutorial-node-exporter-setup

[^39]: https://www.habibza.in/how-to-install-node-exporter-on-ubuntu-18-04

[^40]: https://www.youtube.com/watch?v=peH95b16hNI

[^41]: https://nsrc.org/workshops/2021/sanog37/nmm/netmgmt/en/prometheus/ex-node-exporter.htm

[^42]: https://www.opsramp.com/guides/prometheus-monitoring/prometheus-node-exporter

