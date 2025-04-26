# Monitoring-MySql-App-Using-Prometheus-Grafana

#### Setup Aws Ec2 ( promethus server , mysql app ) 
![image](https://github.com/user-attachments/assets/7a21591d-dfc1-4819-9385-00dffe76b774)

#### ports need to open on Security Group for Prometheus Server
![image](https://github.com/user-attachments/assets/494ed113-9791-4b55-b2db-9864f5132f11)

#### step by step setup & configure promethus server 
commands :
```
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
cd /tmp/
wget https://github.com/prometheus/prometheus/releases/download/v3.3.0-rc.1/prometheus-3.3.0-rc.1.linux-amd64.tar.gz
tar xvf prometheus-3.3.0-rc.1.linux-amd64.tar.gz
cd prometheus-3.3.0-rc.1.linux-amd64/
ls
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo cp prometheus.yml /etc/prometheus/prometheus.yml
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml

- sudo vi /etc/systemd/system/prometheus.service

#### run promothus 
./prometheus 
```
#### Discover a UI  http://<ec2-public-ip>:9090
![image](https://github.com/user-attachments/assets/5ab7d970-dc33-4c33-97e0-6cbb45ec7b3f)


#### running as a systemd :
sudo vi /etc/systemd/system/prometheus.service
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
--storage.tsdb.path /var/lib/prometheus/ 
[Install]
WantedBy=multi-user.target
```
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```
