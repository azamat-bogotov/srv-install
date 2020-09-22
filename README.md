## Первоначальная настройка сервера при установке Ubuntu
---

### Сперва настройки редактора *nano*
В `/etc/nanorc` прописать или расскоментировать значения:
```conf
# set autoident (при копировании и вставке вызывает иногда проблему дополнительного отступа)
set tabsize 4
set tabtospaces
set undo # где Alt+U - undo, Alt+E - redo
```

### Цвет левого промта, вывод текущей ветки в Git
В `~/.bashrc` в конец дописать
```conf
function color_my_prompt {
    local __git_branch='`git branch 2> /dev/null | grep -e ^* | sed -E  s/^\\\\\*\ \(.+\)$/\(\\\\\1\)\ /`'
    local __end='\[\033[00m\]\$ '

    export PS1="\[\033[01;32m\]\u@\[\033[37m\]\h \[\033[01;36m\]\w \[\033[31m\]$__git_branch$__end"
}
color_my_prompt
```

### Системные настройки 

Установка пароля для root

```sh
$ sudo passwd root
```

Настройка сети и перезапуск
```sh
$ sudo nano /etc/network/interfaces
$ sudo ifdown eth0 && sudo ifup eth0
```

#### SSH
- Изменение порта по умолчанию для ssh;
- Отключение возможности подключаться к серверу от пользователя root;
- Рользователи, которые погут подключаться к серверу по ssh;
```sh
$ sudo nano /etc/ssh/sshd_config
```
```conf
Port 2222
PermitRootLogin no
#X11Forwarding yes
#X11DisplayOffset 10
TCPKeepAlive no
MaxStartups 4:30:10
Subsystem sftp internal-sftp
AddressFamily inet
AllowUsers username
ClientAliveCountMax 3
ClientAliveInterval 20
```

Перезапуск сервиса
```sh
$ sudo service ssh restart
```
> ! Необходимо открыть новую сессию и убедиться, что есть подключение после изменения настроек

Обновление репозиториев
```sh
$ sudo apt-get update && sudo apt-get-upgrade
```

Development Tools: gcc, g++, make, zip. 
```sh
$ sudo apt-get install build-essential gcc g++ make zip
```

### nginx
Добавление ключа для доступа к репозиторию *nginx*
```sh
$ cd ~; wget http://nginx.org/keys/nginx_signing.key; sudo apt-key add nginx_signing.key; rm nginx_signing.key
```

Добавление репозиториев для скачивания последней стабильной версии nginx

> Раскомментировать репозитории `canonical` и `ubuntu` в `/etc/apt/sources.list`, дожны быть в конце файла.
>
> там же в `/etc/apt/sources.list` добавить репозитории nginx:
> ```
> deb http://nginx.org/packages/ubuntu/ trusty nginx
> deb-src http://nginx.org/packages/ubuntu/ trusty nginx
>```
> где *trusty* - это кодовое название версии ubuntu, для 14.04 это "trusty", для 12.04 *precise*.
> Обо все этом подробнее [здесь](http://nginx.org/ru/linux_packages.html#stable)

Обновление репозиториев
```sh
$ sudo apt-get update
```

> В случае, если будет ошибка вида: 
```
Ошибка GPG: http://extras.ubuntu.com trusty Release ... 40976EAF437D05B5
```
> то следует добавить ключ
```sh
$ sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 40976EAF437D05B5
```
> заменив *40976EAF437D05B5* на выданное в ошибке значение

Обновление библиотек
```sh
$ sudo apt-get upgrade
```

Увеличение максимально возможного числа открытых файлов (глобально)
> надо прописать в конце файла `/etc/security/limits.conf`:
 ```
* - nofile 1048576
```
- - -

### Добавление пользователя. Создание окружения для сайта. 

Добавление пользователя `username` и домашней директории для него.
> директории, необходимые для разворачивания веб-приложения
> * html - директорий сайта
> * logs - логи nginx, php-fpm
> * tmp  - в будущем будут располагаться файлы сессии
> * conf - расположение настроек nginx для сайта

```sh
$ su root
# adduser username
# usermod -G username -a www-data
# usermod -G www-data -a username

# su username; cd ~; mkdir html; mkdir logs; mkdir tmp; mkdir conf;
```

Удаление пользователя(если произошла ошибка при добавлении)
```sh
$ sudo userdel username
$ sudo rm -rf /home/username
```

Если нужно дать пользователю root привилегии
```sh
$ sudo visudo
```

> При необходимости, добавить возможность пользователю соединяться через ssh
> в `/etc/ssh/sshd_config` в секции *AllowUsers*.

 - - - 

### Установка *nginx*
```sh
$ sudo apt-get install nginx
```

### Настройка nginx.

Открыть для редактирования главный файл настроек nginx: */etc/nginx/nginx.conf* и поменять значения:

```conf
user www-data;
worker_processes 2; # число физических процессорных ядер

events {
    # каждый worker_processes будет обрабатывать worker_connections соединений
    worker_connections 2048;
    use epoll;
}

http {
    #...
    #access_log  /var/log/nginx/access.log  main;
    access_log off;
    #...    
    
    # Отключение вывода версии nginx
    server_tokens off;

    # поставить, если не стоит и убрать другие подключаемые настройки конфигураций
    include /etc/nginx/conf.d/*.conf;
}
```

Заходим в папку `/etc/nginx/conf.d/` и копируем настройки по умолчанию в папку `/home/username/conf/`
```sh
$ cd /etc/nginx/conf.d/
$ sudo cp default.conf /home/username/conf/nginx.conf
$ sudo chown root:root /home/username/conf/nginx.conf
```

Удаляем все конфиги в `/etc/nginx/conf.d/` и делаем ссылку на конфигурацию обратно в `/etc/nginx/conf.d/`
```sh
$ sudo rm -Rf /etc/nginx/conf.d/
$ sudo ln /home/username/conf/nginx.conf /etc/nginx/conf.d/username.conf
```

### Установка php и необходимых модулей 

```sh
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
```

```sh
$ sudo apt-get install php5.6-cli php5.6-common php5.6-mysql php5.6-gd php5.6-fpm php5.6-curl php5.6-json php5.6-mcrypt php5.6-sqlite3 php5.6-tidy php5.6-snmp php5.6-intl

$ sudo apt-get install php7.0-cli php7.0-common php7.0-mysql php7.0-gd php7.0-fpm php7.0-curl php7.0-json php7.0-mcrypt php7.0-sqlite3 php7.0-tidy php7.0-snmp php7.0-intl

$ sudo apt-get install php7.1-cli php7.1-common php7.1-mysql php7.1-gd php7.1-fpm php7.1-curl php7.1-json php7.1-mcrypt php7.1-sqlite3 php7.1-tidy php7.1-snmp php7.1-intl
```

### Установка сервера mysql 5.7
```sh
sudo apt-get install mysql-server-5.7 mysql-client-5.7
```
Добавляем в конце файла `/etc/mysql/my.cnf` (нужные опции можно изменить исходя из требований):
```conf
[client]
port            = 3306
socket          = /var/run/mysqld/mysqld.sock

[mysqld]
open_files_limit = 102400
sql_mode = "STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

slow_query_log = 1
long_query_time = 10
slow_query_log_file = /var/log/mysql/mysql-slow.log

tmp_table_size   = 32M
#key_buffer       = 32M
key_buffer_size  = 32M
sort_buffer_size = 4M
bulk_insert_buffer_size = 32M

default_storage_engine         = InnoDB
innodb_flush_method            = O_DIRECT
innodb_log_files_in_group      = 3
innodb_buffer_pool_size        = 128M
innodb_flush_log_at_trx_commit = 2
```

Если необходимо предоставить доступ извне к mysql-серверу, то в файле `/etc/mysql/mysql.conf.d/mysqld.cnf` необходимо закомментировать строку
```conf
#bind-address		= 127.0.0.1
```

Чтобы не было ошибок при множестве одновременных запросов увеличиваем значение макс. открытых дескрипторов на файлы для mysql с 1024 до 102400 (глобальные настроки не воспринимаются). Для этого добавляем каталог и файл, если их не существуют `/etc/systemd/system/mysql.service.d/nofile.conf`, и прописываем там:
```conf
[Service]
LimitNOFILE=102400
```
После перезапуска mysql, надо убедиться, что настройки изменились `SHOW VARIABLES LIKE 'open_files_limit';`


### настройка php-fpm

Меняем настройки в файлах `/etc/php5/fpm/php.ini` и `/etc/php5/cli/php.ini`

```conf
expose_php = Off
max_execution_time = 30 # max_execution_time = 0 for cli/php.ini
error_reporting = E_ALL | E_STRICT
display_errors = Off
cgi.fix_pathinfo=0
date.timezone = Europe/Moscow
```

В директории `/etc/php5/fpm/pool.d/` создаем копию дефолтного файла настроек *www.conf* -> *poolname.conf* и меняем:

```conf
; pool name (нужно задать уникальное имя пула, можно название проекта)
[project_name]

; Рабочие процессы пула будут работать от имени указанного пользователя и группы
user  = username
group = username

; Пул будет ожидать запросы на указанном Unix-сокете
listen = /var/run/php5-fpm.{project_name}.sock
; или по порту
listen = 127.0.0.1:9000 # свой порт для каждого пула процесов :9001, :9002, ...

; Владелец Unix-сокета, его группа и права доступа к сокету
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

; Динамический менеджер рабочих процессов
pm = dynamic
pm.max_children = 32

; количество дочерних процессов, которые будет создавать PM при старте PHP-FMP (только если pm dynamic);
pm.start_servers = 10

; количество процессов, которые должны оставаться в idle, как “запасные”, ожидая задач на выполнение, если количество меньше – будут созданы новые;
pm.min_spare_servers = 5

; наоборот, максимальное количество процессов, которые должны оставаться в idle, если количество больше – некоторые потомки будут уничтожены;
pm.max_spare_servers = 20

; Тут можно ограничить количество запросов, последовательно обслуживаемых одним процессом
; После этого процесс будет завершён и запущен снова - это может помочь от утечек памяти
pm.max_requests = 1000

; Если обработка одного запроса длится дольше трёх минут - обработка запроса принудительно завершается
request_terminate_timeout = 180s

env[TMP]    = /home/username/tmp
env[TMPDIR] = /home/username/tmp
env[TEMP]   = /home/username/tmp

; можно переопределить значения из php.ini
php_flag[display_errors] = off
php_admin_value[error_log] = /home/username/logs/php-error.log
php_admin_flag[log_errors] = on
php_admin_value[memory_limit] = 32M
php_admin_value[session.save_path] = /home/username/tmp
```

### настройка nginx

В файле `/home/username/conf/nginx.conf` прописываем следующее(с послед. изменением параметров под нужды)

```conf
server {
    listen       80;
    server_name  localhost;

    access_log  off; #/home/username/logs/nginx-access.log;
    root        /home/username/html;
    index       index.php index.html;
    
    # security =========================
    # Отключение вывода версии nginx. В гл. настройках тоже прописано(здесь для независимости)
    server_tokens off;
    
    # запрет на вставку сайта как iframe
    #add_header X-Frame-Options DENY;
    
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block;";
    add_header Strict-Transport-Security "max-age=expireTime";
    
    if ($request_method !~ ^(GET|HEAD|POST)$) {
        return 444;
    }

    if ($http_user_agent ~* (LWP::Simple|BBBike|wget|msnbot|scrapbot|nmap|nikto|wikto|sf|sqlmap|bsqlbf|w3af|acunetix|havij|appscan)) {
        return 403;
    }

    if ($http_referer ~* (babes|forsale|girl|jewelry|love|nudit|organic|poker|porn|sex|teen)) {
        return 403;
    }
    # ==================================

    # Количество и размер буферов для чтения большого заголовка запроса клиента
    large_client_header_buffers 2 1k;
    
    # Таймаут при чтении тела запроса клиента
    client_body_timeout 20;
    
    # gzip
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_min_length 1100;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript; 

    # timeouts. Тестовые настройки
    client_max_body_size 8M;
    client_body_buffer_size 1m;
    #client_body_timeout 15;
    #client_header_timeout 15;
    #keepalive_timeout 2 2;
    #send_timeout 15;
    #sendfile on;
    #tcp_nopush on;
    #tcp_nodelay on;

    location ~ /\. {deny all;}
    #location = /favicon.ico {return 204;}
    #location = /robots.txt {return 204;}

    location / {
        try_files $uri index.php /index.php;
    }

    #location ~ \.php$ {
    #    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    #    fastcgi_pass   unix:/var/run/php5-fpm.{project_name}.sock; # см. настройки php-fpm
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # static files
    location ~* \.(jpg|jpeg|gif|ico|png|htm|html|xml|json|js|css|properties|zip|swf|unity3d)$ {
        expires  max;
    }
}
```

### настройка настройка mysql
> TODO

### перезапуск всех сервисов
```sh
$ sudo service php5-fpm restart
$ sudo service mysql restart
$ sudo service nginx restart
```

### Установка последней версии git
```sh
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt-get update
$ sudo apt-get install git
```
