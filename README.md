# Домашнее задание 1
Задание выполнено с помощью Docker. 
Файл docker-compose приведен.
1. Установлена CMS Wordpress. Для ее работы используются следующие контейнеры: db (mysql), wordpress, webserver (nginx).
2. В контейнерах установлены экспортеры: mysqld-exporter, node-exporter, php-fpm-exporter, blackbox-exporter.
3. Prometheus установлен в контейнере prometheus.
4. Alertmanager установлен в контейнере и настроен на отправку уведомлений в telegram. (По описанию: https://muetsch.io/sending-prometheus-alerts-to-telegram-with-telepush.html)
