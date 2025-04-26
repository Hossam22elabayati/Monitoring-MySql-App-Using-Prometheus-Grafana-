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
```
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
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.enable-lifecycle \
  --web.listen-address="0.0.0.0:9090"
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

:wq
```

```
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```
![image](https://github.com/user-attachments/assets/e332ad45-9308-4d44-96ad-cb6825df1d2e)


### Prometheus and monitor up : 
![image](https://github.com/user-attachments/assets/4f437494-5e92-4752-a977-62321285070b)

#### setup mysql app to add as a target to Promothus : 
### prepare mysql 
```
sudo apt update
sudo apt install mysql-server 
systemctl status mysql.service
```
#### check service : 
![image](https://github.com/user-attachments/assets/683d4754-8969-43e8-b49d-0e615672fcf8)

### install exporter 
```
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.17.2/mysqld_exporter-0.17.2.linux-amd64.tar.gz
tar -xvf mysqld_exporter-0.17.2.linux-amd64.tar.gz
cd mysqld_exporter-0.17.2.linux-amd64/
ls
sudo chmod +x mysqld_exporter
sudo mv mysqld_exporter /usr/local/bin/
mysqld_exporter --version 

```
### enter to mysql create a user to mysql exporter and give permission to access data to provide a metric of  mysql 
```
sudo mysql
>CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY  'StrongPassword'  WITH MAX_USER_CONNECTIONS 2;
>GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost';
> FLUSH PRIVILEGES;
> exit 

 vim /etc/.mysqld_exporter.cnf
#add this lines credentials to mysql 
[client]
user=mysqld_exporter
password=StrongPassword
:wq

sudo chown root:prometheus /etc/.mysqld_exporter.cnf

```

### mysql exporter as a service : 
```
 sudo vim /etc/systemd/system/mysql_exporter.service

[Unit]
Description=Prometheus MySQL Exporter
After=network.target
User=prometheus
Group=prometheus

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/mysqld_exporter \
    --config.my-cnf /etc/.mysqld_exporter.cnf \
    --collect.global_status \
    --collect.info_schema.innodb_metrics \
    --collect.info_schema.processlist \
    --collect.binlog_size \
    --collect.global_variables \
    --collect.info_schema.query_response_time \
    --collect.info_schema.userstats \
    --collect.info_schema.tables \
    --collect.perf_schema.tablelocks \
    --collect.perf_schema.file_events \
    --collect.perf_schema.eventswaits \
    --collect.perf_schema.indexiowaits \
    --collect.perf_schema.tableiowaits \
    --collect.slave_status \
    --web.listen-address=0.0.0.0:9104

[Install]
WantedBy=multi-user.target
:wq

sudo systemctl daemon-reload 
sudo systemctl enable --now mysql_exporter.service
sudo systemctl status mysql_exporter.service
```
### edit a security group 
![image](https://github.com/user-attachments/assets/9b7cdfbe-c83d-40c2-aaac-6b49ffedd820)

### validate is running : 
![image](https://github.com/user-attachments/assets/e4910b58-cad9-420a-b7c3-a3db371f7804)

![image](https://github.com/user-attachments/assets/cddd4d34-94e0-43cc-829e-1e673646eef8)


#### configure mysql-server target to Prometheus-Server 
```
sudo vim /etc/prometheus/prometheus.yml

#add this 

- job_name: "mysql-server"
    static_configs:
      - targets: ["54.82.93.163:9104"]


# for me i added a private ip to instance or public any of them , add your ip 

 sudo systemctl restart prometheus.service


```

### validate 

![image](https://github.com/user-attachments/assets/00e3fa8f-d202-41ef-be70-3d504c89211a)
![image](https://github.com/user-attachments/assets/e5fee025-86f2-40a9-abb1-f1d6bebe4505)


## install Grafana and integrate with Prometheus

### Grafana as a docker container 
install docker first : https://docs.docker.com/engine/install/ubuntu/
```
vim docker-compose.yml

services:
  grafana:
    image: grafana/grafana  # Use grafana/grafana-enterprise for enterprise version
    container_name: grafana
    restart: unless-stopped
    ports:
      - '3000:3000'
    volumes:
      - grafana-storage:/var/lib/grafana

volumes:
  grafana-storage: {}

 sudo docker compose up -d
```

### open grafana :
 your-ip:3000   
username: admin  , 
password: admin

![image](https://github.com/user-attachments/assets/7acf67d9-468f-4144-bfc1-1c2160496764)

### add data source 
![image](https://github.com/user-attachments/assets/c0ac448d-828e-4f0d-8cfe-0f502b21b921)

![image](https://github.com/user-attachments/assets/52095944-229d-4faa-b64b-350f62929f3e)

![image](https://github.com/user-attachments/assets/822be2b9-2a39-42bc-879a-8caff0af11ca)


### import Dashboard for mysql 
url : https://grafana.com/grafana/dashboards/14031-mysql-dashboard/

and
copy id 

![image](https://github.com/user-attachments/assets/bd7b6a3a-daab-44f4-ab0f-31cbcee4b35e)

![image](https://github.com/user-attachments/assets/59af3d7d-2745-425b-a3f1-f127d0d2f7e7)

![image](https://github.com/user-attachments/assets/7cb2fb30-d8c1-46d4-860e-1ded06eda94f)


Dashboard :
![image](https://github.com/user-attachments/assets/4869059c-d301-4cee-baca-85c70cfae817)


# Done 👏👏👏🎉🎉
