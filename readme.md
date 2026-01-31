# Monitoring Mac with Apple Silicon chips with Prometheus and Grafana

This guide describes how to configure a complete monitoring system for Mac with Apple Silicon chips (M1/M2/M3/M4). The system uses Telegraf and mactop on the Mac to collect metrics, Prometheus on a Linux server for storage, and Grafana for visualization.

---

## Architecture

| Component | Host | Port | Function |
|-----------|------|------|----------|
| Telegraf | Mac (YOUR_MAC_IP) | 9273 | System metrics (CPU, RAM, Disk, Network) |
| mactop | Mac (YOUR_MAC_IP) | 9275 | Apple Silicon metrics (GPU, Power, Temp) |
| Prometheus | Linux (YOUR_SERVER_IP) | 9090 | Time-series database |
| Grafana | Linux (YOUR_SERVER_IP) | 3000 | Dashboard and visualization |

> **Note:** Replace `YOUR_MAC_IP` and `YOUR_SERVER_IP` with your actual IP addresses (e.g., 192.168.1.100 and 192.168.1.200)

---

## 1. Installing Prometheus and Grafana on Ubuntu

### 1.1 System Update

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Installing Prometheus

```bash
# Create prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

# Create directories
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus /var/lib/prometheus

# Download Prometheus (check latest version at github.com/prometheus/prometheus)
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-2.48.0.linux-amd64.tar.gz
tar xvf prometheus-2.48.0.linux-amd64.tar.gz
cd prometheus-2.48.0.linux-amd64

# Install binaries
sudo cp prometheus promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool

# Copy configuration
sudo cp -r consoles console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
```

### 1.3 Prometheus Configuration

Create the file `/etc/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'mac-telegraf'
    static_configs:
      - targets: ['YOUR_MAC_IP:9273']

  - job_name: 'mac-mactop'
    static_configs:
      - targets: ['YOUR_MAC_IP:9275']
```

### 1.4 Systemd Service for Prometheus

Create the file `/etc/systemd/system/prometheus.service`:

```ini
[Unit]
Description=Prometheus
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
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

```bash
# Start Prometheus
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

### 1.5 Installing Grafana

```bash
# Add Grafana repository
sudo apt install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | \
  sudo tee /etc/apt/sources.list.d/grafana.list

# Install Grafana
sudo apt update
sudo apt install -y grafana

# Start Grafana
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

> **Note:** Grafana will be accessible at http://YOUR_SERVER_IP:3000 (default credentials: admin/admin)

---

## 2. Installation on Mac (Apple Silicon)

### 2.1 Prerequisites: Homebrew

```bash
# Install Homebrew (if not present)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add to PATH (for Apple Silicon)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 2.2 Installing Telegraf, mactop and screen

```bash
brew install telegraf mactop screen
```

### 2.3 Telegraf Configuration

Create/edit the file `/opt/homebrew/etc/telegraf.conf`:

```toml
[global_tags]

[agent]
  interval = "10s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = ""
  hostname = ""  # Leave empty to use system hostname, or set your preferred name
  omit_hostname = false

# Prometheus output
[[outputs.prometheus_client]]
  listen = ":9273"
  metric_version = 2

# System metrics
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false

[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

[[inputs.diskio]]

[[inputs.mem]]

[[inputs.net]]

[[inputs.processes]]

[[inputs.swap]]

[[inputs.system]]

# SMART disk health
[[inputs.smart]]
  use_sudo = true

# Ping test
[[inputs.ping]]
  urls = ["8.8.8.8"]
  count = 3
  ping_interval = 1.0
  timeout = 2.0
```

### 2.4 Starting Telegraf as a Service

```bash
# Start Telegraf with brew services
brew services start telegraf

# Verify it works
curl http://localhost:9273/metrics
```

### 2.5 Configuring mactop with Prometheus

mactop requires a TTY to work, so we use screen to run it in the background.

Create the LaunchAgent `~/Library/LaunchAgents/com.mactop.prometheus.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.mactop.prometheus</string>
    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/screen</string>
        <string>-S</string>
        <string>mactop</string>
        <string>-dm</string>
        <string>/opt/homebrew/bin/mactop</string>
        <string>-p</string>
        <string>9275</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

```bash
# Load the LaunchAgent
launchctl load ~/Library/LaunchAgents/com.mactop.prometheus.plist

# Verify it works
curl http://localhost:9275/metrics
```

### 2.6 Manual mactop Startup (alternative)

```bash
# To manually start mactop with screen
screen -S mactop -dm /opt/homebrew/bin/mactop -p 9275

# To see the screen session
screen -ls

# To attach to the session
screen -r mactop

# To detach (inside screen): Ctrl+A, then D
```

---

## 3. Configuration Verification

### 3.1 Connectivity Test from Linux Server

```bash
# Test Telegraf
curl http://YOUR_MAC_IP:9273/metrics

# Test mactop
curl http://YOUR_MAC_IP:9275/metrics

# Verify targets in Prometheus
# Open http://YOUR_SERVER_IP:9090/targets in browser
```

### 3.2 Available Metrics from mactop

| Metric | Description |
|--------|-------------|
| `mactop_gpu_usage_percent` | GPU usage percentage |
| `mactop_gpu_freq_mhz` | GPU frequency in MHz |
| `mactop_gpu_temperature_celsius` | GPU temperature |
| `mactop_soc_temp_celsius` | SoC (chip) temperature |
| `mactop_power_watts{component="..."}` | Power consumption (cpu, gpu, dram, system, total) |
| `mactop_cpu_core_usage_percent{type="e\|p"}` | Per-core usage (E-core/P-core) |
| `mactop_ecore_usage_percent` | Average E-cores usage |
| `mactop_pcore_usage_percent` | Average P-cores usage |
| `mactop_thermal_state` | Thermal state (0=Nominal, 1=Fair, 2=Serious, 3=Critical) |
| `mactop_system_info` | System info (labels: model, core_count, gpu_core_count) |

---

## 4. Creating Grafana Dashboard

### 4.1 Adding Prometheus Data Source

1. Go to Configuration > Data Sources > Add data source
2. Select Prometheus
3. URL: http://localhost:9090
4. Click Save & Test

### 4.2 Importing Dashboard

The dashboard JSON file (`macos-apple-silicon-dashboard.json`) can be imported:

1. Go to Dashboards > New > Import
2. Upload the JSON file or paste the content
3. Select the Prometheus data source
4. Click Import

### 4.3 Dashboard Sections

| Section | Content |
|---------|---------|
| System Overview | Uptime, CPU Cores, GPU Cores, RAM, Thermal State, Load Average |
| Apple Silicon - CPU, GPU & Power | CPU/GPU Usage gauges, Temperature gauges, Frequency, Power consumption, Core usage graph |
| Memory | Memory Usage gauge, Breakdown (Wired/Active/Inactive/Free), Swap |
| Disk | Disk Usage, Space Over Time, I/O, SSD Temperature & Health |
| Network | Traffic, Packets, Ping Average, Packet Loss, Latency |

### 4.4 Useful Prometheus Queries

```promql
# CPU Usage (from Telegraf)
100 - cpu_usage_idle{host=~"$host", cpu="cpu-total"}

# GPU Usage (from mactop)
mactop_gpu_usage_percent

# Memory Usage percentage
100 * mem_used{host=~"$host"} / mem_total{host=~"$host"}

# Total Power consumption
mactop_power_watts{component="total"}

# System Load
system_load1{host=~"$host"}
system_load5{host=~"$host"}
system_load15{host=~"$host"}

# Per-core CPU usage (E-cores)
mactop_cpu_core_usage_percent{type="e"}

# Per-core CPU usage (P-cores)
mactop_cpu_core_usage_percent{type="p"}
```

---

## 5. Troubleshooting

### 5.1 mactop doesn't start as a service

mactop requires a TTY to work. The solution is to use screen:

```bash
# Manual startup with screen
screen -S mactop -dm /opt/homebrew/bin/mactop -p 9275

# Verify session
screen -ls
```

### 5.2 Prometheus can't reach the Mac

Verify:

```bash
# On Mac, check firewall
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# If enabled, add exceptions or temporarily disable
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off

# Verify listening ports
netstat -an | grep LISTEN | grep -E "9273|9275"
```

### 5.3 Telegraf doesn't start

```bash
# Check status
brew services list | grep telegraf

# Check logs
tail -f /opt/homebrew/var/log/telegraf.log

# Restart
brew services restart telegraf
```

### 5.4 Dashboard shows no data

1. Verify Prometheus reaches the targets (Status > Targets)
2. Verify the data source is correctly configured
3. Check that dashboard variables ($host, $datasource) are set

---

## 6. Maintenance

### 6.1 Updating Components

```bash
# On Mac
brew update && brew upgrade telegraf mactop screen

# On Linux
sudo apt update && sudo apt upgrade grafana
# For Prometheus, download new version and replace binaries
```

### 6.2 Configuration Backup

```bash
# Mac
cp /opt/homebrew/etc/telegraf.conf ~/backup/
cp ~/Library/LaunchAgents/com.mactop.prometheus.plist ~/backup/

# Linux
sudo cp /etc/prometheus/prometheus.yml ~/backup/
# Export Grafana dashboard as JSON from web interface
```

---

## License

MIT

---

*Configuration tested on Mac mini M4 Pro with macOS 26 Tahoe
