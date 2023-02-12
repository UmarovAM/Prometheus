# Prometheus

### Install alertmanager on prometheus 
    wget 
    https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz

    tar -xvf alertmanager-*linux-amd64.tar.gz

### Скопируйте содержимое архива в папки:

    cp ./alertmanager-*.linux-amd64/alertmanager /usr/local/bin
    cp ./alertmanager-*.linux-amd64/amtool /usr/local/bin
    cp ./alertmanager-*.linux-amd64/alertmanager /usr/local/bin
    cp ./alertmanager-*.linux-amd64/amtool /usr/local/bin
    cp ./alertmanager-*.linux-amd64/alertmanager.yml /etc/prometheus
    chown -R prometheus:prometheus /etc/prometheus/alertmanager.yml

### Сервис для работы с Node Exporter
    nano /etc/systemd/system/prometheus-alertmanager.service
[Unit]
Description=Alertmanager Service
After=network.target
[Service]
EnvironmentFile=-/etc/default/alertmanager
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml 
--storage.path=/var/lib/prometheus/alertmanager $ARGS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
### Пропишите автозапуск:
     systemctl enable prometheus-alertmanager
     systemctl start prometheus-alertmanager
     systemctl status prometheus-alertmanager

### 
