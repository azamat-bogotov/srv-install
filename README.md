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

Изменение порта по умолчанию для ssh и отключение возможности подключаться к серверу от пользователя root
```sh
$ nano /etc/ssh/sshd_config
```

Обновление репозиториев
```sh
$ apt-get update && apt-get-upgrade
```

Development Tools: gcc, g++, make. 
```sh
$ sudo apt-get install gcc g++ make
# либо
$ sudo apt-get install build-essential
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

### Создание окружения для сайта. 

Добавление пользователя и домашней директории для пользователя `test`
```sh
$ su root
$ adduser test
$ usermod -G test -a www-data
$ usermod -G www-data -a test

$ su test; cd ~; mkdir html; mkdir logs; mkdir tmp; mkdir conf;
```

> * html   - директорий сайта
> * logs   - логи nginx, php-fpm
> * tmp    - в будущем будут располагаться файлы сессии
> * conf   - расположение настроек nginx для сайта

Если нужно дать пользователю root привилегии
```sh
$ visudo
```

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
    worker_connections 2048; # каждый worker_processes будет обрабатывать worker_connections соединений
    use epoll;
}

http {
    #...
    #access_log  /var/log/nginx/access.log  main;
    access_log off;
    #...    
    
    # Отключение вывода версии nginx
    server_tokens off;

    # Количество и размер буферов для чтения большого заголовка запроса клиента
    large_client_header_buffers 2 1k;

    # Таймаут при чтении тела запроса клиента
    client_body_timeout 20;

    include /etc/nginx/conf.d/*.conf; # поставить, если не стоит и убрать другие подключаемые настройки конфигураций
}
```

Заходим в папку `/etc/nginx/conf.d/` и копируем настройки по умолчанию в папку `/home/test/conf/`
```sh
$ cd /etc/nginx/conf.d/
$ sudo cp default.conf /home/test/conf/nginx.conf
$ sudo chown root:root /home/test/conf/nginx.conf
```

Удаляем все конфиги в `/etc/nginx/conf.d/` и делаем ссылку на конфигурацию обратно в `/etc/nginx/conf.d/`
```sh
$ sudo rm -Rf /etc/nginx/conf.d/
$ sudo ln /home/test/conf/nginx.conf /etc/nginx/conf.d/test.conf
```

### Настройка конфигурационного файла для сайта. 

Меняем настройки `/home/test/conf/nginx.conf`

> TODO

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


### настройка nginx

### настройка настройка mysql

### перезапуск всех сервисов
```sh
$ sudo service php5-fpm restart
$ sudo service mysql restart
$ sudo service nginx restart
```
