# EC2 Instance Monitoring Setup using Prometheus, Blackbox Exporter, Grafana, and Node Exporter

## Overview
This setup will monitor a Jenkins server using Prometheus, Blackbox Exporter, Grafana, and Node Exporter. We will create two dashboards:
1. **Node Exporter Dashboard**: Monitors system-level metrics of the Jenkins server.
2. **Blackbox Exporter Dashboard**: Monitors the endpoint availability of the WebGoat application.

---

## Step 1: Launch a Monitoring EC2 Instance
- Create an EC2 instance (Ubuntu 22.04 recommended).
- Allow inbound security group rules for Prometheus, Grafana, Node Exporter, and Blackbox Exporter.
- Install required dependencies:
  ```sh
  sudo apt update && sudo apt install -y wget curl unzip
  ```

---

## Step 2: Install Node Exporter
1. Download and install Node Exporter in Jenkins EC2 instance:
   ```sh
   wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
   tar xvf node_exporter-linux-amd64.tar.gz
   ```
2. Start Node Exporter:
   ```sh
   ./node_exporter &
   ```

---

## Step 3: Install Blackbox Exporter
1. Download and install Blackbox Exporter in Monitoring EC2 Instance:
   ```sh
   wget https://github.com/prometheus/blackbox_exporter/releases/latest/download/blackbox_exporter-linux-amd64.tar.gz
   tar xvf blackbox_exporter-linux-amd64.tar.gz
   sudo mv blackbox_exporter-*/blackbox_exporter /usr/local/bin/
   ```
2. Start Blackbox Exporter:
   ```sh
   ./blackbox_exporter &
   ```
---

## Step 4: Install Prometheus in Monitoring EC2 Instance
1. Download and extract Prometheus:
   ```sh
   wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
   tar xvf prometheus-linux-amd64.tar.gz
   cd prometheus-3.1.0.linux-amd64
   ```
   
4. Edit Prometheus config file:
   ```sh
   sudo nano /etc/prometheus/prometheus.yml
   ```
   **Example Configuration:**
   ```yaml
   global:
     scrape_interval: 15s

   scrape_configs:
     - job_name: 'node exporter'
       static_configs:
         - targets: ['jenkins_instance_public_ip:9100']

     - job_name: 'jenkins'
       metrics_path: '/prometheus'
       static_configs:
         - targets: ['jenkins_instance_public_ip:8080']

     # Blackbox exporter configuration
     - job_name: 'blackbox'
       metrics_path: /probe
       params:
         module: [http_2xx]  # Look for a HTTP 200 response.
       static_configs:
         - targets:
           - http://webgoat_instance_public_ip/WebGoat
       relabel_configs:
         - source_labels: [__address__]
           target_label: __param_target
         - source_labels: [__param_target]
           target_label: instance
         - target_label: __address__
           replacement: monitoring_instance_public_ip:9115
   ```
5. Start Prometheus in Background:
   ```sh
   ./prometheus &
   ```

---

## Step 5: Install Grafana in Monitoring EC2 Instance
1. Install Grafana:
   ```sh
   sudo apt install -y grafana
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```
2. Access Grafana at `http://your-ec2-ip:3000`
3. **Add Prometheus as a Data Source:**
   - Navigate to **Configuration > Data Sources** in Grafana.
   - Click **Add data source** and select **Prometheus**.
   - Enter the Prometheus URL (e.g., `http://monitoring_instance_public_ip:9090`) and save.

---

## Step 6: Create Dashboards
### **1. Node Exporter Dashboard**
- Import a Node Exporter dashboard from Grafana Labs (Dashboard ID: `1860`).
- Set Prometheus as the data source.

### **2. Blackbox Exporter Dashboard**
- Import a Node Exporter dashboard from Grafana Labs (Dashboard ID: `7587`).
- Set Prometheus as the data source.

---

## Step 7: Install Prometheus Plugin in Jenkins
1. Navigate to **Manage Jenkins > Manage Plugins**.
2. Install the **Prometheus Plugin**.
3. Restart Jenkins.
4. Enable Prometheus metrics in Jenkins:
   - Go to **Manage Jenkins > Configure System**.
   - Enable **Expose Prometheus metrics** under the Prometheus section.
   - Save and restart Jenkins.

---



