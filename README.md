# Домашнее задание к занятию "Система мониторинга Zabbix" - Нигматуллин Ильяс

## Задание 1

Установлен Zabbix Server с веб-интерфейсом на Ubuntu 24.04 с использованием PostgreSQL и Apache.  
После установки выполнена первичная настройка базы данных, импорт схемы и запуск сервисов.  
Веб-интерфейс Zabbix успешно открылся, выполнен вход в административную панель.

### Этапы выполнения

1. Обновил систему и установил необходимые утилиты.
2. Установил PostgreSQL.
3. Подключил официальный репозиторий Zabbix 7.4.
4. Установил пакеты Zabbix Server, Zabbix Frontend, Apache и Zabbix Agent.
5. Создал пользователя и базу данных PostgreSQL для Zabbix.
6. Импортировал начальную схему Zabbix в базу данных.
7. Настроил файл `/etc/zabbix/zabbix_server.conf`.
8. Настроил locale для корректной работы веб-интерфейса.
9. Запустил и добавил в автозапуск службы Zabbix Server, Zabbix Agent и Apache.
10. Открыл веб-интерфейс и выполнил вход в административную панель.

![Скриншот Zabbix Dashboard](img/zabbix-dashboard.png)

### Использованные команды

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y ca-certificates wget gnupg lsb-release
sudo apt install -y postgresql

wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_latest_7.4+ubuntu24.04_all.deb
sudo apt update

sudo apt install -y zabbix-server-pgsql zabbix-frontend-php php8.3-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent

sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix

zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

sudo nano /etc/zabbix/zabbix_server.conf

sudo apt install -y locales
sudo locale-gen en_US.UTF-8
sudo update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
sudo nano /etc/apache2/envvars

sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```md

## Задание 2

Установлен Zabbix Agent на два хоста:
- `Zabbix server` — сервер Zabbix на локальной Ubuntu VM
- `yc-agent` — Debian 11 VM в Yandex Cloud

Оба хоста были добавлены в Zabbix Server в раздел `Data collection -> Hosts`.  
После настройки агентов и добавления хостов данные начали поступать в раздел `Monitoring -> Latest data`.

### Этапы выполнения

1. На локальном сервере был проверен установленный и работающий `zabbix-agent`.
2. В Yandex Cloud была создана отдельная VM с Debian 11.
3. На второй VM был установлен `zabbix-agent`.
4. В конфигурационном файле `/etc/zabbix/zabbix_agentd.conf` были указаны разрешённый сервер и имя хоста.
5. Агент был запущен и добавлен в автозагрузку.
6. Хост `yc-agent` был добавлен в веб-интерфейсе Zabbix с шаблоном `Linux by Zabbix agent`.
7. После настройки сетевого доступа и проверки соединения данные от второго хоста начали отображаться в Zabbix.

### Использованные команды

```bash
sudo apt update
sudo apt install -y ca-certificates wget gnupg lsb-release

wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian11_all.deb
sudo dpkg -i zabbix-release_latest_7.4+debian11_all.deb
sudo apt update
sudo apt install -y zabbix-agent

sudo nano /etc/zabbix/zabbix_agentd.conf
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
systemctl status zabbix-agent --no-pager

sudo ss -ltnp | grep 10050
sudo tail -n 30 /var/log/zabbix/zabbix_agentd.log
