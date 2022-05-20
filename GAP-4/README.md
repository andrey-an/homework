# Домашняя работа №4
1. Установлен ELK стек в docker;
2. Создано правило пересылки лога /var/log/secure для sshd с помощью rsync на порт udp 10514 (конфиги для проверки: `rsyslog.d/`);
3. Создан конфиг для logstash для приема логов по порту udp 10514 (конфиг logstash для проверки: `logstash_pipeline/rsyslog.conf`);
4. Создан индекс в elastic (результат проверки в `images/index_check.png`);
5. Создан дашборд в kibana (результат для проверки: `images/kibana_dashboard.png`)
