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
