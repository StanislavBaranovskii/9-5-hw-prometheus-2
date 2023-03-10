# Домашнее задание к занятию "`9.5. Prometheus. ч.2`" - `Барановский Станислав`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

## Задание 1

Создайте файл с правилом оповещения, как в лекции, и добавьте его в конфиг Prometheus.

*Погасите node exporter, стоящий на мониторинге, и прикрепите скриншот раздела оповещений Prometheus, где оповещение будет в статусе Pending.*

Создаём файл с правилом оповещения
```
sudo nano /etc/prometheus/netology-test.yml
sudo chown -R prometheus:prometheus /etc/prometheus/netology-test.yml
```
Содержимое файла с правилом оповещения
```
groups: # Список групп
- name: netology-test # Имя группы
  rules: # Список правил текущей группы
  - alert: InstanceDown # Название текущего правила
    expr: up == 0 # Логическое выражение
    for: 1m # Сколько ждать отбоя предупреждения перед отправкой оповещения
    labels:
      severity: critical # Критичность события
    annotations: # Описание
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.' # Полное описание але>
      summary: Instance {{ $labels.instance }} down # Краткое описание алерта
```
Подключаем правила к Prometheus
```
sudo nano /etc/prometheus/prometheus.yml #Содержимое файла ниже в блоке кода
sudo systemctl restart prometheus
sudo systemctl status prometheus
sudo systemctl status node-exporter
sudo systemctl stop node-exporter
```
Содержимое файла prometheus.yml
```
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

rule_files:
  - "netology-test.yml"

scrape_configs:
  - job_name: "prometheus"
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090', 'localhost:9100']
```
http://127.0.0.1:9090

![Скриншот раздела оповещений Prometheus](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-1.png "Скриншот раздела оповещений Prometheus")

---

## Задание 2

Установите Alertmanager и интегрируйте его с Prometheus.

*Прикрепите скриншот Alerts из Prometheus, где правило оповещения будет в статусе Fireing, и скриншот из Alertmanager, где будет видно действующее правило оповещения.*
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-386.tar.gz
tar -xvf alertmanager-0.25.0.linux-386.tar.gz 
sudo cp alertmanager-0.25.0.linux-386/alertmanager /usr/local/bin
sudo cp alertmanager-0.25.0.linux-386/amtool /usr/local/bin
sudo cp alertmanager-0.25.0.linux-386/alertmanager.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/alertmanager.yml
sudo nano /etc/systemd/system/prometheus-alertmanager.service #Содержимое файла ниже в блоке кода
sudo systemctl enable prometheus-alertmanager
sudo systemctl start prometheus-alertmanager
sudo systemctl status prometheus-alertmanager
sudo journalctl -xeu prometheus-alertmanager.service
```
Содержимое файла prometheus-alertmanager.service
```
[Unit]
Description=Alertmanager Service
After=network.target
[Service]
EnvironmentFile=-/etc/default/alertmanager
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/alertmanager \
--config.file=/etc/prometheus/alertmanager.yml \
--storage.path=/var/lib/prometheus/alertmanager $ARGS
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
Подключаем Prometheus к Alertmanager
```
sudo nano /etc/prometheus/prometheus.yml #В секцию alerting -> alertmanagers -> static_configs -> static_configs вписываем строку - localhost:9093
sudo systemctl restart prometheus.service
sudo systemctl status prometheus.service
```
http://localhost:9093

![Скриншот Alerts из Prometheus](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-2-1.png "Скриншот Alerts из Prometheus")
![Скриншот из Alertmanager](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-2-2.png "Скриншот из Alertmanager")

---

## Задание 3

Активируйте экспортёр метрик в Docker и подключите его к Prometheus.

*Приложите скриншот браузера с открытым эндпоинтом, а также скриншот списка таргетов из интерфейса Prometheus.*
```
# Устанавливаем Docker
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/pgp | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" > /etc/apt/sources.list.d/docker.list
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo docker run hello-world
sudo systemctl status docker

# Включаем выгрузку данных на хосте с Docker
sudo nano /etc/docker/daemon.json #Содержимое файла ниже в блоке кода
sudo systemctl restart docker
sudo systemctl status docker

```
Содержимое файла daemon.json
```
{
 "metrics-addr" : "localhost:9323",
 "experimental" : true
}
```
http://localhost:9323/metrics

```
# Добавляем endpoint Docker в Prometheus
sudo nano /etc/prometheus/prometheus.yml #В секцию  scrape_configs-> static_configs -> targets добавляем адрес localhost:9323
sudo systemctl restart prometheus.service
sudo systemctl status prometheus.service
sudo systemctl start node-exporter.service
sudo systemctl status node-exporter.service
```

![Скриншот браузера с открытым эндпоинтом](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-3-1.png "Скриншот браузера с открытым эндпоинтом")
![Скриншот списка таргетов из интерфейса Prometheus](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-3-2.png "Скриншот списка таргетов из интерфейса Prometheus")

---

## Задание 4*

Создайте свой дашборд Grafana с различными метриками Docker и сервера, на котором он стоит.

*Приложите скриншот, на котором будет дашборд Grafana с действующей метрикой.*
```
```
![Дашборд Grafana с действующей метрикой](https://github.com/StanislavBaranovskii/9-5-hw-prometheus-2/blob/main/img/9-5-4.png "Дашборд Grafana с действующей метрикой")

---
