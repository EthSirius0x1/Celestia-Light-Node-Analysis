# Celestia-Light-Node-Analysis
## Performance Analysis of Celestia Light Node
- Node ID:    12D3KooWLQrtS9VB28akua2BxXLkQz5GW9sfBEUGLKQsKpSPaSTx

Introduction:
Monitoring the performance of your node is crucial to ensuring your application runs smoothly and efficiently. In this blog post, we'll discuss how to set up Prometheus and Node Exporter on an Ubuntu server to monitor your node's CPU usage, memory usage, and bandwidth usage.

Prerequisites:

- A VPS running Ubuntu
- A running node in the folder /root/.celestia-light-blockspacerace-0

## Step 1: Install Prometheus and Node Exporter
 - Download and extract the latest Prometheus release:
``` 
wget https://github.com/prometheus/prometheus/releases/download/v2.35.1/prometheus-2.35.1.linux-amd64.tar.gz
tar xvf prometheus-2.35.1.linux-amd64.tar.gz
```
- Move the Prometheus binaries and set proper permissions:
```
sudo cp prometheus-2.35.1.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.35.1.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

- Download and extract the latest Node Exporter release:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvf node_exporter-1.3.1.linux-amd64.tar.gz
```

- Move the Node Exporter binary and set proper permissions:
```
sudo cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

## Step 2: Configure Prometheus and Node Exporter

- Create the Prometheus configuration file /etc/prometheus/prometheus.yml:
```
# Global configuration
global:
  scrape_interval: 15s

# A scrape configuration for running Prometheus on the same host.
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

- Create the systemd service files for Prometheus and Node Exporter:

For Prometheus (/etc/systemd/system/prometheus.service):
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
For Node Exporter (/etc/systemd/system/node_exporter.service):
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
- Reload the systemd configuration and start both services:
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl start node_exporter
```
- Enable both services to start on boot:
```
sudo systemctl enable prometheus
sudo systemctl enable node_exporter
```

## Step 3: Analyze Node Performance Metrics

Open your browser and navigate to http://<your_server_ip>:9090 to access the Prometheus web interface.

Use the following PromQL queries to analyze your node's performance:

CPU usage:
```
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```
![Ekran Resmi 2023-04-07 02 50 05](https://user-images.githubusercontent.com/107340835/230518721-9204cbb0-9c40-40a9-a56d-519d97a4f944.png)



Memory usage:
```
node_memory_MemUsed_bytes / node_memory_MemTotal_bytes * 100
```
![Ekran Resmi 2023-04-07 02 49 12](https://user-images.githubusercontent.com/107340835/230518727-a3417d1d-da70-4812-9080-6b16e568bd9f.png)


Network bandwidth:
```
rate(node_network_receive_bytes_total[5m]) + rate(node_network_transmit_bytes_total[5m])
```
![Ekran Resmi 2023-04-07 02 50 56](https://user-images.githubusercontent.com/107340835/230518738-b0df3a3c-3eb4-46d1-bddc-dee0da171e06.png)



## Conclusion:

In this blog post, we've covered how to set up Prometheus and Node Exporter on an Ubuntu server to analyze your node's hardware performance. By monitoring CPU usage, memory usage, and network bandwidth, you can gain valuable insights into your Celestia node's performance and make necessary adjustments to optimize it.
