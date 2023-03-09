# Prometheus

## Установка Prometheus
```     
#Добавьте пользователя prometheus
    useradd --no-create-home --shell /bin/false prometheus

#Последнюю версию найдите на GitHub
    wget 
    https://github.com/prometheus/prometheus/releases/download/v2.40.1/prometheus-2.40.1.linux-386.tar.gz

#Извлеките архив и скопируйте файлы в необходимые директории:
    tar xvfz prometheus-2.28.1.linux-amd64.tar.gz
    cd prometheus-2.28.1.linux-amd64
    mkdir /etc/prometheus
    mkdir /var/lib/prometheus
    cp ./prometheus promtool /usr/local/bin/
    cp -R ./console_libraries /etc/prometheus
    cp -R ./consoles /etc/prometheus
    cp ./prometheus.yml /etc/prometheus

#Передайте права на файлы пользователю Prometheus:
    chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus 
    chown prometheus:prometheus /usr/local/bin/prometheus
    chown prometheus:prometheus /usr/local/bin/promtool

#Запустите и проверьте результат:
    /usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --web.console.templates=/etc/prometheus/consoles --web.console.libraries=/etc/prometheus/console_libraries

#Создание сервис для работы с Prometheus
    nano /etc/systemd/system/prometheus.service

    [Unit]
    Description=Prometheus Service myPrometheus
    After=network.target
    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
    ExecReload=/bin/kill -HUP $MAINPID Restart=on-failure
    [Install]
    WantedBy=multi-user.target

#Передайте права на файл:
    chown -R prometheus:prometheus /var/lib/prometheus

#Запустите prometheus
    sudo systemctl enable prometheus
    sudo systemctl start prometheus
    sudo systemctl status prometheus
```
## Установка Node Exporter
```
#Скачайте архив с Node Exporter и извлеките его:
    wget 
    https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.
    tar.gz
    tar xvfz node_exporter-*.*-amd64.tar.gz

#Перейдите в появившуюся папку:
    cd node_exporter-*.*-amd64
    ./node_exporter 

#Скопируйте Node Explorer в папку Prometheus
    mkdir /etc/prometheus/node-exporter
    cp ./* /etc/prometheus/node-exporter

#Передайте права на папку пользователю Prometheus
    chown -R prometheus:prometheus /etc/prometheus/node-exporter/

#Создайте сервис для работы с Node Explorer
    nano /etc/systemd/system/node-exporter.service

#Вставьте в файл сервиса следующее содержимое:
    [Unit]
    Description=Node Exporter Lesson 9.4
    After=network.target
    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/etc/prometheus/node-exporter/node_exporter
    [Install]
    WantedBy=multi-user.target 

#Тестирование сервиса Node Explorer
    sudo systemctl enable node-exporter
    sudo systemctl start node-exporter
    sudo systemctl status node-exporter
```

## Добавление Node Exporter в Prometheus
```
#Отредактируйте конфигурацию Prometheus:
    nano /etc/prometheus/prometheus.yml

#Добавьте в scrape_config адрес экспортера:
    scrape_configs:
    — job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
    — targets: ['localhost:9090', 'localhost:9100'] 

#Перезапустите Prometheus:
    systemctl restart prometheus
```
## Установка Grafana
```
#Скачайте и установите DEB-пакет:
    wget https://dl.grafana.com/oss/release/grafana_9.2.4_amd64.deb
    dpkg -i grafana_9.2.4_amd64.deb

#Включите автозапуск и запускаем сервер Grafana:
    systemctl enable grafana-server 
    systemctl start grafana-server 
    systemctl status grafana-server

#Проверьте статус подключившись на адрес: 
   https://<наш сервер>:3000  Стандартный логин и пароль admin \ admin

#Добавление Dashboard в Grafana
    Перейдите в раздел Configuration > Data Sources и нажмите на 
    Add data sourcе В появившемся списке выберите Prometheus
    http://localhost:9090
    Нажмите кнопку Save & Test
    На сайте grafana.com найдите нужный Dashboard и к скопируйте его 
    ID.
    Перейдите в раздел [+] > Import и в поле Import via grafana.com 
    внесите скопированный ID или ссылку на Dashboard.
    В выпадающем списке VictoriaMetrics выберите Prometheus
    Нажмите кнопку Import
```
## Install alertmanager on prometheus 
```
    wget 
    https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
    tar -xvf alertmanager-*linux-amd64.tar.gz

#Скопируйте содержимое архива в папки:
    cp ./alertmanager-*.linux-amd64/alertmanager /usr/local/bin
    cp ./alertmanager-*.linux-amd64/amtool /usr/local/bin
    cp ./alertmanager-*.linux-amd64/alertmanager.yml /etc/prometheus
    chown -R prometheus:prometheus /etc/prometheus/alertmanager.yml

# Сервис для работы с Node Exporter
    nano /etc/systemd/system/prometheus-alertmanager.service
    
    [Unit]
    Description=Alertmanager Service
    After=network.target
    [Service]
    EnvironmentFile=-/etc/default/alertmanager
    User=prometheus
    Group=prometheus
    Type=simple
    ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml --storage.path=/var/lib/prometheus/alertmanager $ARGS
    ExecReload=/bin/kill -HUP $MAINPID
    Restart=on-failure
    [Install]
    WantedBy=multi-user.target

#Пропишите автозапуск:
     systemctl enable prometheus-alertmanager
     systemctl start prometheus-alertmanager
     systemctl status prometheus-alertmanager

#Добавьте в сonfig-файл Prometheus подключение к Alertmanager:
     nano /etc/prometheus/prometheus.yml
    
     alerting:
       alertmanagers:
         - static_configs:
           - targets: # Можно указать как "targets: [‘localhost:9093’]"
             - localhost:9093
    sudo systemctl restart prometheus
    systemctl status prometheus
             
```             
## Создайте файл с правилом оповещения:
```
    nano /etc/prometheus/test.yml

    groups: # Список групп
    - name: test # Имя группы
      rules: # Список правил текущей группы
      - alert: InstanceDown # Название текущего правила
        expr: up == 0 # Логическое выражение
        for: 1m # Сколько ждать отбоя предупреждения перед отправкой оповещения
        labels:
          severity: critical # Критичность события
        annotations: # Описание
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.' # Полное описание алерта
          summary: Instance {{ $labels.instance }} down # Краткое описание алерта

#Подключение правила к Prometheus
    nano /etc/prometheus/prometheus.yml

#Добавьте в раздел rule_files запись:
    - "test.yml"

    systemctl restart prometheus
    systemctl status prometheus

# Настройка оповещений в Alertmanager
    nano /etc/prometheus/alertmanager.yml

    global:
    route:
      group_by: ['alertname'] # Параметр группировки оповещений - по имени
      group_wait: 30s # Сколько ждать восстановления, перед тем как отправить первое оповещение.
      group_interval: 10m # Сколько ждать перед тем как дослать оповещение о новых сработках по текущему алерту.
      repeat_interval: 60m # Сколько ждать перед тем как отправить повторное оповещение
      receiver: 'email' # Способ которым будет доставляться текущее оповещение
    receivers: # Настройка способов оповещения
    - name: 'email' 
      email_configs:
      - to: 'yourmailto@todomain.com'
        from: 'yourmailfrom@fromdomain.com'
        smarthost: 'mailserver:25'
        auth_username: 'user'
        auth_identity: 'user'
        auth_password: 'paS$w0rd'


# Проверка оповещений Alertmanager
После внесения всех изменений перезапустите Alertmanager и выключите экспортер, стоящий на сервере Prometheus:

    sudo systemctl restart prometheus-alertmanager
    systemctl status prometheus-alertmanager
    sudo systemctl stop node-exporter
    systemctl status node-exporter

#Теперь можете проверить интерфейсы Prometheus и Alertmanager, расположенные на стандартных портах 9090 и 9093

```
# Мониторинг Docker в Prometheus

```
#Docker из коробки поддерживает мониторинг с помощью Prometheus. Для того чтобы включить выгрузку данных на хосте с Docker, нужно создать файл daemon.json:
     nano /etc/docker/daemon.json

#Добавить 
     {
    "metrics-addr" : "ip_нашего_сервера:9323", # пишем 0.0.0.0
    "experimental" : true
    }

#Перезапустите Docker:

    systemctl restart docker && systemctl status docker
    Для проверки можно открыть адрес http://server_ip:port/metrics.
```
## Добавление endpoint Docker в Prometheus. 
```
Чтобы поставить только что организованный endpoint на мониторинг, необходимо отредактировать файл prometheus.yml:

    nano /etc/prometheus/prometheus.yml

В раздел static_configs добавьте новый endpoint. 
   
#пример:
      static_configs:
      - targets: ["localhost:9090", "localhost:9100", "localhost:9323"]  

#Вид раздела #1
    static_configs:
    - targets: ['localhost:9090', 'localhost:9100', 'server_ip:9323']
       
 #Вид раздела #2
    static_configs:
    - targets:
      - localhost:9090
      - localhost:9100
      - server_ip:9323
      
#Перезапустите Prometheus
     systemctl restart prometheus
```
      
## Настройка GRAFANA для DOCKER
```
#Настройка собственного Dashboard
#В интерфейсе Grafana нажмите на “+” и выберите 
Dashboards
#Нажмите на + New dashboard > Add new panel
#В выпадающем меню Metrics выберите: engine > 
engine_daemon_container_states_containers;
#Нажмите на Apply и перейдите в интерфейс Dashboard
#Сохраните Dashboard
```
![image](https://user-images.githubusercontent.com/118117183/218322672-e31203ed-eec6-45d3-9c3c-2ef10add849c.png)

