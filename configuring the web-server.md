## Разворачиваем  Nginx + PHP-FPM+ MySQL На Ubuntu 18.04

### Установка Ngix

В качестве базы данных мы будем ставить MariaDB. Установка все необходимых пакетов.

```sudo apt install --no-install-recommends nginx mariadb-server php php-fpm php-zip php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc```

После установки нам надо установить пароль пользователю root в базу, т.к. по умолчанию он пустой. Так надо для безопасности.

```sudo mysql -u root```

 Зададим ieyah7Aew6oom8ohsooghies:
 
  ```set password for 'root'@'localhost' = PASSWORD('ieyah7Aew6oom8ohsooghies');```
  
  ```exit```
  
После окончания процедуры установки можно проверить запущены ли у нас установленные сервисы. Используем для этого комаду ***systemctl***

Пр. ```systemctl status nginx```

Теперь нам нужно настроить Nginx для работы с Wordpress. Для нашего сайта будем использовать этот движок. 

Главный конфиг Nginx лежит тут ***/etc/nginx/nginx.conf*** 

Давай посмотрим его содержимое. 

Чтобы вывести содержимое файла на экран в Linux есть команда ***cat***:

```cat /etc/nginx/nginx.conf```
 
В файле nginx.conf строка include показывает нам, что удобнее создавать для каждого сайта  свой конфиг. Nginx подгрузит все содержимое папки /etc/nginx/conf.d/*conf в один большой конфигурационный файл. Он сделает это сам. А мы создадим для каждого сайта свой конфиг.

Закрывайте nginx.conf нажав Ctrl-X

Переходим в директорию conf.d

Скачиваем готов конфиг: ```sudo wget http://distrib.sipteco.com/website.conf```

Веб-сервер может обрабатывать запросы на множество сайтов, доменных имен. 

Поле server_name нужно вебсерверу чтобы распределять входящие запросы и понимать какой запрос какому сайту отправить.

В нашем случае доменного имени для сайта пока нет. Но есть IP адрес у виртуалки. Впишем IP адрес в поле server_name. Не забудь точку с запятой.

Теперь давай создадим дирректорию где будет лежать наш сайт. Ее путь указан в конфиге. 

Давайте проверим все ли мы правильно настроили.

```sudo nginx -t```

Теперь загрузим наш измененный конфиг в память nginx: ```sudo systemctl reload nginx```

Создадим дирректорию  командой: ```sudo mkdir -p /var/www/ubuntu-wordpress.ru/```

и дадим права nginx читать этот файл.: ```sudo chown -R www-data:www-data /var/www/```

Nginx мы настроили. Теперь давайте настроим PHP-FPM. (это тот который обрабатывает сам PHP) 

Его конфиги лежат тут /etc/php/7.2/fpm/pool.d/

удалим стандартный и создадим свой. 

cd /etc/php/7.2/fpm/pool.d/

Yдаляем файл ***www*** ***.*** ***conf***

Скачаем файл конфига: ```wget http://distrib.sipteco.com/ubuntu-wp.conf```

Проверим все ли окей: ```sudo systemctl reload php7.2-fpm```

Перезапустим php-fpm: ```sudo systemctl restart php7.2-fpm```

Проверим статус: ```sudo systemctl  status php7.2-fpm```

Давай теперь проверим работает эта связка у нас или нет

создаим в каталоге тестовый файлик где у нас должен лежать сайт (помнишь в nginx мы прописывали в конфиге папку /var/www/ubuntu-wordpress.ru ) 

Командой touch можем создать файлик 

touch /var/www/ubuntu-wordpress.ru/index.php

nano /var/www/ubuntu-wordpress.ru/index.php

Вставляем в файл:

phpinfo

<?php

phpinfo();

phpinfo(INFO_MODULES);

?>




