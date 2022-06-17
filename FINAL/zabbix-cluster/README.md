# Zabbix HA with PostgreSQL Cluster

## Цель
Создать стенд с кластером высокой доступности [Zabbix](https://www.zabbix.com/documentation/current/ru/manual/concepts/server/ha), кластером PostgreSQL и поставить стенд на мониторинг в Prometheus.   


## Описание
Стенд собран на основе контейнеров Docker.   
Высокая доступность [Zabbix](https://www.zabbix.com/documentation/current/ru/manual/concepts/server/ha) выполнена за счет встроенной функциональности.   
В PostgreSQL настроена репликация с помощью [repmgr](https://repmgr.org/) и производится балансировка экземпляров с помощью [pgpool2](https://github.com/pgpool/pgpool2).   
Балансировка между Zabbix frontend осуществляется с помощью [HAProxy](http://www.haproxy.org/).

Все компоненты кластера Zabbix стоят на мониторинге в [Prometheus](https://prometheus.io/).   
Проверяется только доступность компонентов.   
В [Prometheus](https://prometheus.io/) настроены правила, оповещения отправляются через [alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/).

В Grafana есть дашборд состояния компонентов кластера (Zabbix-Cluster).   
[В Zabbix может быть настроена генерация отчетов](https://www.zabbix.com/documentation/current/ru/manual/web_interface/frontend_sections/reports/scheduled), компонент `zabbix-web-service` на стенде есть.

## Архитектура
Архитектура стенда представлена на изображении ниже.    

![Архитектура](https://raw.githubusercontent.com/andrey-an/homework/main/FINAL/zabbix-cluster/images/zabbix-cluster.png "Архитектура")

### Описание компонентов

|Компонент|Роль|Образ|Мониторинг|Способ мониторинга|Директория|Файл переменных окружения|
|---|---|---|---|---|---|---|
|alertmanager|События из Prometheus|prom/alertmanager:latest|---|---|alertmanager|---|
|blackbox-exporter|Проверки http, tcp|[prom/blackbox-exporter](https://github.com/prometheus/blackbox_exporter)|---|---|blackbox-config|---|
|grafana|Система визуализации|grafana/grafana:latest|---|---|grafana_provisioning|---|
|haproxy|Балансировщик|haproxy:latest|Prometheus|blackbox-exporter http:80|haproxy|---|
|pg-0|СУБД Zabbix primary|[bitnami/postgresql-repmgr:14](https://github.com/bitnami/bitnami-docker-postgresql-repmgr)|Prometheus|blackbox-exporter tcp:5432|data|env_vars/.env_pg|
|pg-1|СУБД Zabbix standby|[bitnami/postgresql-repmgr:14](https://github.com/bitnami/bitnami-docker-postgresql-repmgr)|Prometheus|blackbox-exporter tcp:5432|data|env_vars/.env_pg|
|pgpool|Балансировщик PostgreSQL|[bitnami/pgpool:4](https://github.com/bitnami/bitnami-docker-pgpool)|Prometheus|pgpool2-exporter|pgpool|env_vars/.env_pgpool2|
|pgpool2-exporter|Мониторинг pgpool2|[pgpool/pgpool2_exporter](https://github.com/pgpool/pgpool2_exporter)|---|---|---|env_vars/.env_pgpool2_exporter|
|prometheus|Система мониторинга|[quay.io/prometheus/prometheus:latest](https://quay.io/repository/prometheus/prometheus)|---|---|prometheus|---|
|zabbix-server-1|Zabbix Backend|[zabbix-server-pgsql:ol-6.0-latest](https://github.com/zabbix/zabbix-docker)|Prometheus|blackbox-exporter tcp:10051|zbx_env|env_vars/.env_db_pgsql, env_vars/.env_srv|
|zabbix-server-2|Zabbix Backend|[zabbix-server-pgsql:ol-6.0-latest](https://github.com/zabbix/zabbix-docker)|Prometheus|blackbox-exporter tcp:10051|zbx_env|env_vars/.env_db_pgsql, env_vars/.env_srv|
|zabbix-web-1|Zabbix Frontend|[zabbix-web-nginx-pgsql:ol-6.0-latest](https://github.com/zabbix/zabbix-docker)|Prometheus|blackbox-exporter http:8080|zbx_env|env_vars/.env_db_pgsql, env_vars/.env_web|
|zabbix-web-2|Zabbix Frontend|[zabbix-web-nginx-pgsql:ol-6.0-latest](https://github.com/zabbix/zabbix-docker)|Prometheus|blackbox-exporter http:8080|zbx_env|env_vars/.env_db_pgsql, env_vars/.env_web|
|zabbix-web-service|Сервис генерации отчетов Zabbix|[zabbix-web-service:ol-6.0-latest](https://github.com/zabbix/zabbix-docker)|---|---|zbx_env|env_vars/.env_web_service|

### Схема docker-compose

Схема docker-compose сгенерирована с помощью [pmsipilot/docker-compose-viz](https://github.com/pmsipilot/docker-compose-viz).
![Схема docker-compose](https://raw.githubusercontent.com/andrey-an/homework/main/FINAL/zabbix-cluster/images/docker-compose.png "Схема docker-compose")

## Интерфейсы
|Компонент|Интерфейс|Порт|
|---|---|---|
|Grafana|Web|3000|
|Prometheus|Web|9090|
|Zabbix|Web|80|

## Установка
Все файлы переменных окружения заполнены стандартными данными.   
При установке необходимо указать свои данные (логины, пароли и т.д.) в соответствующих файлах окружения (приведены в таблице выше) в директории `env_vars`.   
Конфигурационные файлы для компонентов содержатся в соответствующих директориях (приведены в таблице выше).    

Запуск контейнеров:
```sh
docker-compose up -d
```
### Подключение к PostgreSQL
Для подключения к PostgreSQL можно использовать PostgreSQL client в контейнере:
```sh
docker run -it --rm \
  --network zabbix-cluster_zbx-network \
  bitnami/postgresql:14 \
  psql -h pgpool -U zabbix -d zabbix
```
Для проверки статусов репликации:
```sh
docker exec -ti pg-0 \
/opt/bitnami/scripts/postgresql-repmgr/entrypoint.sh \
repmgr -f /opt/bitnami/repmgr/conf/repmgr.conf \
cluster show
```
### Настройка оповещений из Prometheus Alertmanager
Для тестирования и более быстрой настройки оповещений в Telegram, можно использовать [Telepush](https://muetsch.io/sending-prometheus-alerts-to-telegram-with-telepush.html).   
Оповещения проходят по следующему пути:
```sh
Prometheus --> Alertmanager --> Telepush API --> Telegram
```
Для настройки оповещений:
1. Откройте новый чат с [TelepushBot](https://t.me/MiddlemanBot) и напишите `/start`. Вы получите _токен_;
2. Вставьте _токен_ в конфигурационный файл `alertmanager/config.yml` в разделе `receivers: name: 'telepush'` в атрибут `url`.   

Пример уведомления в Telegram представлен на изображении ниже.    
![Оповещение из Prometheus Alertmanager](https://raw.githubusercontent.com/andrey-an/homework/main/FINAL/zabbix-cluster/images/telegram-alert.png "Оповещение из Prometheus Alertmanager")
### Визуализация
В Grafana есть дашборды для просмотра статусов компонентов:
- Zabbix-Cluster - статусы компонентов кластеров, на основе проверок из Prometheus;
- [PGPool2](https://grafana.com/grafana/dashboards/15805) - дашборд для [pgpool2_exporter](https://github.com/pgpool/pgpool2_exporter).   

Изображение дашборда `Zabbix-Cluster` из Grafana представлено ниже.
![Zabbix-Cluster](https://raw.githubusercontent.com/andrey-an/homework/main/FINAL/zabbix-cluster/images/grafana-dashboard.png "Zabbix-Cluster")
