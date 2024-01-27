# Grafana, Prometheus and Node Exporter Setup

## Install Grafana
### 1) Download the Latest Grafana Version
```
sudo apt update
```

```
sudo apt-get install -y adduser libfontconfig1 musl
```
```
wget https://dl.grafana.com/enterprise/release/grafana-enterprise_10.3.1_amd64.deb
```
```
sudo dpkg -i grafana-enterprise_10.3.1_amd64.deb
```

### 2) Start the Service

```
systemctl daemon-reload
```
```
systemctl start grafana-server
```
```
systemctl status grafana-server
```
```
sudo systemctl enable grafana-server.service
```
### 3) Access Grafana Web Interface

Open security port 3000
```
<ip_address>:3000
```


## Install Prometheus

### 1) Create a System User for Prometheus

```
sudo groupadd --system prometheus
```

```
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
### 2) Create Directories for Prometheus
```
sudo mkdir /etc/prometheus
```

```
sudo mkdir /var/lib/prometheus
```

### 3) Download Prometheus and Extract Files
```
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```

```
tar vxf prometheus*.tar.gz
```

### 4) Navigate to the Prometheus Directory
``` 
cd prometheus*/
```

## Configuring Prometheus

### 1) Move the Binary Files & Set Owner
```
sudo mv prometheus /usr/local/bin
```
```
sudo mv promtool /usr/local/bin
```
```
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```
```
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

### 2) Move the Configuration Files & Set Owner
```
sudo mv consoles /etc/prometheus
```
```
sudo mv console_libraries /etc/prometheus
```
```
sudo mv prometheus.yml /etc/prometheus
```
```
sudo chown prometheus:prometheus /etc/prometheus
```
```
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
```

```
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```
```
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
### 3) Set the targets
```
sudo nano /etc/prometheus/prometheus.yml
```
### 4) Create Prometheus Systemd Service and add the following code inside
```
sudo nano /etc/systemd/system/prometheus.service
```

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

### 5) Reload Systemd
```
sudo systemctl daemon-reload
```

### 6) Start Prometheus Service
```
sudo systemctl enable prometheus
```
```
sudo systemctl start prometheus
```
### 7) Check Prometheus Status
```
sudo systemctl status prometheus
```
### 8) Access Prometheus Web Interface
Open security port 9090
```
localhost:9090 or <ip_address>:9090
```
## Download Node Exporter
### 1) Download the Node Exporter binary 
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
```
### 2) Create User
```
sudo groupadd -f node_exporter
```
```
sudo useradd -g node_exporter --no-create-home --shell /bin/false node_exporter
```
```
sudo mkdir /etc/node_exporter
```
```
sudo chown node_exporter:node_exporter /etc/node_exporter
```
### 3) Unpack Node Exporter Binary
```
tar -xvf node_exporter-1.0.1.linux-amd64.tar.gz
```

```
mv node_exporter-1.0.1.linux-amd64 node_exporter-files
```
### 4) Install Node Exporter
```
sudo cp node_exporter-files/node_exporter /usr/bin/
```
```
sudo chown node_exporter:node_exporter /usr/bin/node_exporter
```
### 5) Setup Node Exporter Service
```
sudo vi /usr/lib/systemd/system/node_exporter.service
```
Add the following code
```
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/guides/node-exporter/
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
ExecStart=/usr/bin/node_exporter \
  --web.listen-address=:9200

[Install]
WantedBy=multi-user.target
```
```
sudo chmod 664 /usr/lib/systemd/system/node_exporter.service
```

### 6) Reload systemd and Start Node Exporter
```
sudo systemctl daemon-reload
```
```
sudo systemctl start node_exporter
```
```
sudo systemctl status node_exporter
```
```
sudo systemctl enable node_exporter.service
```

### 7) Verify Node Exporter is Running

Open security port 9200 
```
http://<ip-address>:9200/metrics
```
### 8) Add target in the prometheus
Add the following code inside the prometheus.yml 
```
 - job_name: "node_exporter"

      # metrics_path defaults to '/metrics'
      # scheme defaults to 'http'.

    static_configs:
      - targets: ["<ip-address>:9200"]

```