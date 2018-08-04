# bitrix_dockerized_env
Окружение для запуска битрикс через docker. 

Требования к установленному софту:
 - docker (`curl -sSL https://get.docker.com/ | sh`)
 - docker-compose (```sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose```)

Для отправки почты в контейнере с apache и PHP используется ssmtp - настройки SMTP (подходят для Яндекс.ПДД) 
задаются в переменных окружения в docker-compose.yml. Реализовано выполнение скрипта bitrix/modules/main/tools/cron_events.php 
через интервалы, которые задаются в переменной CRON_INTERVAL_SEC. 

Для запуска установить переменные окружения в файле docker-compose.yml:
 - CRON_INTERVAL_SEC - интервал, с которым будут выполняться агенты (запуск скрипта cron_events.php)
 - SMTP_USER=имя пользователя SMTP
 - SMTP_PASS=пароль SMTP
 - SMTP_REWRITE_DOMAIN=домен в pdd.yandex.ru
 - SMTP_HOSTNAME=хост, который резолвится в DNS для данного хоста
 - MYSQL_PASSWORD=желаемый пароль к БД для пользователя bitrix
 - MYSQL_ROOT_PASSWORD=желаемый пароль к БД для пользователя root

`docker-compose build && docker-compose up -d`

затем открыть http://host:8000/restore.php

# Примечание

По умолчанию файлы Битрикс сохраняются в папку www от пользователя:группы 33 (www-data). Если на хостовой системе необходим доступ 
к файлам от имени локального пользователя, то 
используются переменные среды APACHE_RUN_USER и APACHE_RUN_GROUP. Для проброса локальных имён пользователей в контейнер с Apache используйте доступ
к файлам /etc/passwd и /etc/group (добавьте их в раздел volumes для сервиса apache).

```
  apache:
    build: ./apache
    ports:
      - 8000:80
    volumes:
      - ./www:/var/www/html
      - /etc/passwd:/etc/passwd:ro
      - /etc/group:/etc/group:ro
    links:
      - mysql
    restart: always
    ulimits:
      stack: -1
    environment:
      - CRON_INTERVAL_SEC=60
      - SMTP_MAILHUB=smtp.yandex.ru:465
      - SMTP_USER=server@pdd-host.ru
      - SMTP_PASS=password
      - SMTP_REWRITE_DOMAIN=pdd-host.ru
      - SMTP_HOSTNAME=www.host.ru
      - APACHE_RUN_USER=localuser
      - APACHE_RUN_GROUP=localgroup
```
