# Домашнее задание 2
Задание выполнено с помощью Docker. 
Файл docker-compose приведен.
1. Использована установленная ранее CMS Wordpress. Для ее работы используются следующие контейнеры: db (mysql), wordpress, webserver (nginx). В контейнерах установлены экспортеры: mysqld-exporter, node-exporter, php-fpm-exporter, blackbox-exporter. Prometheus установлен в контейнере prometheus. Grafana установлена в контейнере.
2. Создан дашборд в Grafana "Infra / Node Exporter Full". Данный дашборд создан на основе данных от node-exporter из Prometheus. Использован готовый шаблон дашборда из репозитория Grafana c id 1860.
3. Создан дашборд в Grafana "app / Wordpress Perfomance". Дашборд содержит сводные данные с экспортеров: mysqld-exporter, php-fpm-exporter, blackbox-exporter. Отображает работу компонентов CMS WordPress.

Скриншоты расположены в папке GAP-2.

