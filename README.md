## Monitoring with Prometheus

Create two VM's with the Security Group with inbound rules as:  
Ports required to be open for VM's
Custom TCP	TCP	587	0.0.0.0/0  
HTTP	TCP	80	0.0.0.0/0  
Custom TCP	TCP	30000 - 32767	0.0.0.0/0  
HTTPS	TCP	443	0.0.0.0/0  
Custom TCP	TCP	6443	0.0.0.0/0  
Custom TCP	TCP	27017	0.0.0.0/0  
SMTP	TCP	25	0.0.0.0/0  
SMTPS	TCP	465	0.0.0.0/0  
SSH	TCP	22	0.0.0.0/0  
sustom TCP	TCP	3000 - 10000	0.0.0.0/0  


For every VM do sudo apt update at the start  

##### For VM1 (Node Exporter)  
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz  
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz  

##### For VM2 (Prometheus, Alertmanager, Blackbox Exporter)
Prometheus  
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz  
tar xvfz prometheus-2.52.0.linux-amd64.tar.gz  

Alertmanager  
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz  
tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz  

Blackbox Exporter  
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz  
tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz  


##### For VM1  
get the website running   
In our case its a Java App. clone the github link.  cd into the directory  
type java in  shell. It will show the command for downloading java.  
Once java is downloaded, cd into target.  type java -jar jar_name. website will be up on port 8080  
 
cd into node_exporter and start node_exporter service  
./node_exporter &  

##### For VM2

###### Prometheus Configuration (`prometheus.yml`)

###### Global Configuration
```yaml
global:
  scrape_interval: 15s                # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s            # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
```

###### Alertmanager Configuration
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'          # Alertmanager endpoint
```

######  Rule Files
```yaml
rule_files:
   - "alert_rules.yml"                # Path to alert rules file
  # - "second_rules.yml"              # Additional rule files can be added here
```

######  Scrape Configuration
######  Prometheus Itself
```yaml
scrape_configs:
  - job_name: "prometheus"            # Job name for Prometheus

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]   # Target to scrape (Prometheus itself)
```

###### Node Exporter
```yaml
  - job_name: "node_exporter"         # Job name for node exporter

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["<ip>:9100"]  # Target node exporter endpoint
```

###### Blackbox Exporter
```yaml
  - job_name: 'blackbox'              # Job name for blackbox exporter
    metrics_path: /probe              # Path for blackbox probe
    params:
      module: [http_2xx]              # Module to look for HTTP 200 response
    static_configs:
      - targets:
        - http://prometheus.io        # HTTP target
        - https://prometheus.io       # HTTPS target
        - http://<ip>:8080/  # HTTP target with port 8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <ip>:9115  # Blackbox exporter address
```