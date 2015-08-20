## Первоначальная настройка сервера при установке Ubuntu
---

### Сперва настройки редактора *nano*
В `/etc/nanorc` прописать или расскоментировать значения:
```conf
set autoident
set tabsize 4
set tabtospaces
set undo # где Alt+U - undo, Alt+E - redo
```

### Системные настройки 

Установка пароля для root

```sh
$ passwd root
```

Настройка сети и перезапуск
```sh
$ sudo nano /etc/network/interfaces
$ sudo /etc/init.d/networking restart
```

#### SSH
- Изменение порта по умолчанию для ssh;
- Отключение возможности подключаться к серверу от пользователя root;
- Рользователи, которые погут подключаться к серверу по ssh;
```sh
$ nano /etc/ssh/sshd_config
```
```conf
Port 2222
PermitRootLogin no
AddressFamily inet
AllowUsers username
```

Перезапуск сервиса
```sh
$ service ssh restart
```

Обновление репозиториев
```sh
$ apt-get update && apt-get-upgrade
```

Development Tools: gcc, g++, make, zip. 
```sh
$ sudo apt-get install gcc g++ make zip
# либо
$ sudo apt-get install build-essential zip
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
$ apt-get update
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
$ apt-get upgrade
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
$ adduser username
$ usermod -G username -a www-data
$ usermod -G www-data -a username

$ su username; cd ~; mkdir html; mkdir logs; mkdir tmp; mkdir conf;
```

Удаление пользователя(если произошла ошибка при добавлении)
```sh
$ userdel username
$ rm -rf /home/username
```

Если нужно дать пользователю root привилегии
```sh
$ visudo
```

> При необходимости, добавить возможность пользователю соединяться через ssh
> в `/etc/ssh/sshd_config` в секции *AllowUsers*.

 - - - 

### Установка *nginx*
```sh
$ apt-get install nginx
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
> модули php5-suhosin php5-apc не находит

```sh
$ sudo apt-get install php5-cli php5-common php5-mysql php5-gd php5-fpm php5-cgi php5-mcrypt php5-curl php5-json

$ sudo apt-get install php-pear php5-sqlite php5-redis php5-memcached php5-tidy php5-xmlrpc php5-xsl php5-mhash php5-pspell php5-snmp
```

### Установка сервера mysql 5.6
```sh
sudo apt-get install mysql-server-5.6 mysql-client-5.6
```

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
