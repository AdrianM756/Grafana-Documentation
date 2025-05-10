## What is Grafana?

[Grafana]([https://github.com/grafana/grafana](https://grafana.com/)) is an open and composable observability and data visualization platform. It allows you to query, visualize, alert on and understand your metrics no matter where they are stored.

## Collect Metrics

Before we can start visualizing our data, we need to collect some metrics. The easiest way to do this is by collecting some system metrics on our virtual environments. To do this, we will use [Node Exporter](https://prometheus.io/docs/guides/node-exporter/#monitoring-linux-host-metrics-with-the-node-exporter) and [Prometheus](https://prometheus.io/). Lets break down each of these components and we can install them within our virtual enviroment.
<br>

## Node Exporter

Node Exporter is a Prometheus exporter for hardware and OS metrics exposed by *nix systems, written in Go with pluggable metric collectors.It allows for the collection of hardware and OS metrics and is a great way to collect system metrics for your virtual enviroments.
<br>

***NOTE:*** While the Prometheus Node Exporter is for unix systems, there is the [Windows exporter]() for Windows that serves an analogous purpose.
<br>

## Installing Node Exporter

To install Node Exporter, we will used the command:
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
```
<br>

Extract the [tarball](https://wiki.debian.org/TarBall) and navigate to the extracted directory. Move to bin directory.
```
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz && cd node_exporter-1.7.0.linux-amd64 && sudo cp node_exporter /usr/local/bin
```
<br>

## Extractong the tarball

Extract the tarball and navigate to the extracted directory. Move to bin directory.
```
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
```

```
cd node_exporter-1.7.0.linux-amd64
```

```
sudo cp node_exporter /usr/local/bin
```
<br>

## Setting up Node Exporter Service

Create an ```node_exporter``` service file.
```
sudo nano /usr/lib/systemd/system/node_exporter.service
```
<br>

Then add the following configurations:
```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
<br>

Change the permission of the file.
```
sudo chmod 664 /usr/lib/systemd/system/node_exporter.service
```
<br>

## Reloading systemd and starting Node Exporter

Reload the systemd service to register and start the Node Exporter service.
```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
```
<br>

Once it is running, try to access it via ```localhost``` using it's default port ```9100```. You should be able to get this dashboard:

![image](https://github.com/user-attachments/assets/89922c8d-139b-4b33-b552-f538724d0561)
<br>

##Prometheus

Prometheus is a monitoring and alerting toolkit that is designed for reliability, scalability, and maintainability. It is a tool that is use for collecting and querying metrics and is a great way to store the metrics collected by Node Exporter.

## Installing Prometheus

To install Node Prometheus, we will used the command:
```
wget https://github.com/prometheus/prometheus/releases/download/v2.50.1/prometheus-2.50.1.linux-amd64.tar.gz
```
<br>

We will then extract the tarball and navigate to the extracted directory. Move to bin directory.
```
tar -xvf prometheus-2.50.1.linux-amd64.tar.gz
```

```
cd prometheus-2.50.1.linux-amd64 
```

```
sudo cp prometheus /usr/local/bin
```
<br>

We will then create, copy, and modify some users and files.
```
sudo useradd -rs /bin/false prometheus
```

```
sudo mkdir /etc/prometheus /var/lib/prometheus 
```

```
sudo cp prometheus.yml /etc/prometheus/prometheus.yml 
```

```
sudo cp -r consoles/ console_libraries/ /etc/prometheus/ 
```

```
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```
<br>

We will then paste the following configurations  on the ```/etc/systemd/system/prometheus.service``` file:
```
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
<br>

## Reloading systemd and starting Prometheus

Reload the systemd service to register and start the Prometheus service.
```
sudo systemctl daemon-reload
sudo systemctl start prometheus
```
<br>


Next, We will add the Node Exporter as a target to Prometheus. Here is what the ```prometheus.yml``` file should look like:
```
# my global config
global:
  scrape_interval: 5s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 5s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "Node Exporter"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["0.0.0.0:9100"]
```
<br>

We will then restart the prometheus service. Once it goes back, try to access prometheus via ```localhost:9090```. You should be a able to get this dashboard:

![image](https://github.com/user-attachments/assets/1bbf8e26-3a40-4a0e-912e-687b4a9b3e3e)
<br>

## Installing Grafana

To install Grafana, We will be using the following commands:
```
sudo apt-get install -y apt-transport-https software-properties-common wget
```

```
sudo mkdir -p /etc/apt/keyrings/
```

```
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
```
<br>

On the ```/etc/apt/sources.list.d/grafana.list``` file, paste the following commands:
```
deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main
```
<br>

Next, We will then update our apt repository and install grafana.
```
sudo apt-get update && sudo apt-get install -y grafana
```
<br>

## Staring the Grafana Service

To start the Grafana service, we will use the command:
```
sudo systemctl start grafana-server
```
<br>

To check the status, we can use the command:
```
sudo systemctl status grafana-server
```
<br>

Let's try to access it via localhost using it's default port ```3000```.
```
http://localhost:3000
```
<br>

You should be a able to get this:

![image](https://github.com/user-attachments/assets/ea791949-bf24-40b9-9c9f-012bf0f2cfa5)
<br>

To access it, we can use the default credentials ```admin:admin```. It will then prompt it's dashboard:

![image](https://github.com/user-attachments/assets/5423aabc-7c56-4cce-9dfa-5539357ad1d3)





