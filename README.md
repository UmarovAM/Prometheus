# Prometheus
### Установка Prometheus
    useradd --no-create-home --shell /bin/false prometheus
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
    ExecStart=/usr/local/bin/alertmanager --config.file=/etc/prometheus/alertmanager.yml --storage.path=/var/lib/prometheus/alertmanager $ARGS
    ExecReload=/bin/kill -HUP $MAINPID
    Restart=on-failure
    [Install]
    WantedBy=multi-user.target
### Пропишите автозапуск:
     systemctl enable prometheus-alertmanager
     systemctl start prometheus-alertmanager
     systemctl status prometheus-alertmanager

### Добавьте в сonfig-файл Prometheus подключение к Alertmanager:
     nano /etc/prometheus/prometheus.yml
     alerting:
       alertmanagers:
         - static_configs:
           - targets: # Можно указать как "targets: [‘localhost:9093’]"
             - localhost:9093
### Создайте файл с правилом оповещения:
    nano /etc/prometheus/netology-test.yml
    groups: # Список групп
    - name: netology-test # Имя группы
      rules: # Список правил текущей группы
      - alert: InstanceDown # Название текущего правила
        expr: up == 0 # Логическое выражение
        for: 1m # Сколько ждать отбоя предупреждения перед отправкой оповещения
        labels:
          severity: critical # Критичность события
        annotations: # Описание
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.' # Полное описание алерта
          summary: Instance {{ $labels.instance }} down # Краткое описание алерта
### Подключение правила к Prometheus
    nano /etc/prometheus/prometheus.yml
    # Добавьте в раздел rule_files запись:
    - "netology-test.yml"
    systemctl restart prometheus
    systemctl status prometheus
### Настройка оповещений в Alertmanager
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
### Проверка оповещений Alertmanager

После внесения всех изменений перезапустите Alertmanager и выключите экспортер, стоящий на сервере Prometheus:

    sudo systemctl restart prometheus-alertmanager
    systemctl status prometheus-alertmanager
    sudo systemctl stop node-exporter
    systemctl status node-exporter
Теперь можете проверить интерфейсы Prometheus и Alertmanager, расположенные на стандартных портах 9090 и 9093

# Мониторинг Docker в Prometheus

### Docker из коробки поддерживает мониторинг с помощью Prometheus. Для того чтобы включить выгрузку данных на хосте с Docker, нужно создать файл daemon.json:
     nano /etc/docker/daemon.json
     # добавить 
     {
    "metrics-addr" : "ip_нашего_сервера:9323", # пишем 0.0.0.0
    "experimental" : true
    }
Перезапустите Docker:

    systemctl restart docker && systemctl status docker
    
  Для проверки можно открыть адрес http://server_ip:port/metrics

### Добавление endpoint Docker в Prometheus
Чтобы поставить только что организованный endpoint на 
мониторинг, необходимо отредактировать файл prometheus.yml:

    nano /etc/prometheus/prometheus.yml
    #В раздел static_configs добавьте новый endpoint. 

Вид раздела #1

    static_configs:
    - targets: ['localhost:9090', 'localhost:9100', 'server_ip:9323']
Вид раздела #2

    static_configs:
    - targets:
      - localhost:9090
      - localhost:9100
      - server_ip:9323


     systemctl restart prometheus
пример:

    nano /etc/prometheus/prometheus.yml  
 
    static_configs:
      - targets: ["localhost:9090", "localhost:9100", "localhost:9323"]  
      
Перезапустите Prometheus

### set GRAFANA

![image](https://user-images.githubusercontent.com/118117183/218322672-e31203ed-eec6-45d3-9c3c-2ef10add849c.png)

