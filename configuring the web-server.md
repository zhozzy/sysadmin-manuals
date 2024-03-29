## Разворачиваем  Nginx + PHP-FPM+ MySQL На Ubuntu 18.04

### Установка MariaDB

В качестве базы данных мы будем ставить MariaDB. Установка все необходимых пакетов.

```sudo apt install --no-install-recommends nginx mariadb-server php php-fpm php-zip php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc```

После установки нам надо установить пароль пользователю root в базу, т.к. по умолчанию он пустой. Так надо для безопасности.

```
sudo mysql -u root

set password for 'root'@'localhost' = PASSWORD('ieyah7Aew6oom8ohsooghies');
  
exit
```
Создадим базу данных для Wordpress: ```create database wp_mysitedb;```
 
Создадим пользователя wpuser1 с паролем samplepassword: ```CREATE USER 'wpuser1'@'localhost' IDENTIFIED BY 'samplepassword';```

Важно знать, что после собаки указывает адрес с которого вы будете подключаться. localhost - это такой виртуальный IP адрес и имя вашей виртуалки. Он всегда есть в системе и равен 127.0.0.1. Если у вас есть софт, который работает по протоколу IP, но все софты находятся на одном компьютере, то взаимодействие происходит через localhost. Если вдруг база данных будет на одном сервере, а веб-сервер на другом, то нужно создавать пользователя wpuser1@<адрес с которого обращается веб сервер>

Дадим пользвателю wpuser1 права на чтение и запись к базе, которую мы создали: ```GRANT ALL PRIVILEGES ON  wp_mysitedb. * TO 'wpuser1'@'localhost';```

Применим настройки: ```FLUSH PRIVILEGES;```
  
После окончания процедуры установки можно проверить запущены ли у нас установленные сервисы. Используем для этого комаду ***systemctl***

Пр. ```systemctl status nginx```

### Установка Nginx

Теперь нам нужно настроить Nginx для работы с Wordpress. Для нашего сайта будем использовать этот движок. Главный конфиг Nginx лежит тут ***/etc/nginx/nginx.conf*** . Давай посмотрим его содержимое. Чтобы вывести содержимое файла на экран в Linux есть команда ***cat***: ```cat /etc/nginx/nginx.conf```
 
В файле nginx.conf строка include показывает нам, что удобнее создавать для каждого сайта  свой конфиг. Nginx подгрузит все содержимое папки /etc/nginx/conf.d/*conf в один большой конфигурационный файл. Он сделает это сам. А мы создадим для каждого сайта свой конфиг.

Закрывайте nginx.conf нажав Ctrl-X

Переходим в директорию conf.d

Скачиваем готовый конфиг: ```sudo wget http://distrib.sipteco.com/website.conf```

Веб-сервер может обрабатывать запросы на множество сайтов, доменных имен. Поле **server_name** нужно вебсерверу чтобы распределять входящие запросы и понимать какой запрос какому сайту отправить. В нашем случае доменного имени для сайта пока нет. Но есть IP адрес у виртуалки. Впишем **IP адрес** в поле server_name. Не забудь точку с запятой.

Теперь давай создадим дирректорию где будет лежать наш сайт. Ее путь указан в конфиге. Давайте проверим все ли мы правильно настроили:

```sudo nginx -t```

Теперь загрузим наш измененный конфиг в память nginx: ```sudo systemctl reload nginx```

Создадим дирректорию  командой: ```sudo mkdir -p /var/www/ubuntu-wordpress.ru/```

и дадим права nginx читать этот файл.: ```sudo chown -R www-data:www-data /var/www/```

### Установка PHP

Nginx мы настроили. Теперь давайте настроим PHP-FPM. (это тот который обрабатывает сам PHP). Его конфиги лежат тут **/etc/php/7.2/fpm/pool.d/**. Удалим стандартный и создадим свой. 

```cd /etc/php/7.2/fpm/pool.d/```

Yдаляем файл ***www*** ***.*** ***conf***

Скачаем файл конфига: ```wget http://distrib.sipteco.com/ubuntu-wp.conf```

Проверим все ли окей: ```sudo systemctl reload php7.2-fpm```

Перезапустим php-fpm: ```sudo systemctl restart php7.2-fpm```

Проверим статус: ```sudo systemctl  status php7.2-fpm```

Давай теперь проверим работает эта связка у нас или нет. Создадим в каталоге тестовый файлик где у нас должен лежать сайт (помнишь в nginx мы прописывали в конфиге папку /var/www/ubuntu-wordpress.ru ) 

Командой touch можем создать файлик 

```touch /var/www/ubuntu-wordpress.ru/index.php```

```nano /var/www/ubuntu-wordpress.ru/index.php```

Вставляем в файл:

```phpinfo

<?php

phpinfo();

phpinfo(INFO_MODULES);

?>
```
### Установка Wordpress

Установка Wordpress сводится к простому “далее-далее-готово” поэтому скачиваем с официального сайта архив с самим движком:

https://ru.wordpress.org/download/ или прямая ссылка https://ru.wordpress.org/latest-ru_RU.tar.gz

Перейдем в каталог  /var/www/ubuntu-wordpress.ru/

```cd /var/www/ubuntu-wordpress.ru/```

и скачаем в него архив с wordpress, файл называется latest-ru_RU.tar.gz

```sudo wget https://ru.wordpress.org/latest-ru_RU.tar.gz```

В linux нет "расширений" у файлов как в windows. Но окончание tar.gz подсказывает нам, что этот файл является архивом, который запакован упаковщиком Tar (он просто собирает несколько файлов в один без сжатия) и сжат архиватором Gzip. Распакуем при помощи tar . Tar сам запустит gzip; Для этого мы укажем ключ -z


ключ -x говорит команде tar чтомы распаковываем, а не запаковываем

ключ -v  показывает результат работы (verbose)

ключ -f говорит о том что мы распаковываем именно файл с архивом

```tar -xzvf latest-ru_RU.tar.gz```

Все распаковалось в папку wordpress. 

Скопируем содержимое папки wordpress в каталог /var/www/ubuntu-wordpress.ru

```sudo cp -rp wordpress/* /var/www/ubuntu-wordpress.ru/```

***cp*** - команда копирования 

ключ -r говорит что надо скопировать рекурсивнно - значит все что лежит внутри
-p ключ говорит сохранить все права на файл не перезаписывая их 

А звездочка в конце wordpress/* говорит, что надо сначала скопировать именно файлы внутри   wordpress. Если написать без звездочки sudo cp -rp wordpress/* /var/www/ubuntu-wordpress.ru/, то cp попробует скопировать папку wordpress в /var/www/ubuntu-wordpress.ru/ и выдаст ошибку, потому что там уже есть такая папка

Дадим nginx права на эту папку, чтобы он мог читать и исполнять то что в ней содержится

```chown -R www-data:www-data /var/www/ubuntu-wordpress.ru```

Открываем Chrome и вводим IP адрес нашей виртуалки. Нажимаем “Вперёд!”, заполняем на следующей странице все формы данными от mysql - база, пользователь, пароль:

Заканчиваем настройку сайта указанием пароля для админки и имени сайта:

Ну все. Можешь отдавать Wordpress нашим разработчикам)))) 

Зайди в админку по адресу виртуалка/wp-admin

***password wordpress admin FXQb3rMABxxfjxE***

![GeneralWebSite](https://user-images.githubusercontent.com/65624428/149674745-c2c7810a-2bfa-479c-a56f-57d8083ea470.png)


