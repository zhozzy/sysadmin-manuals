## Настройка ATS Asterisk на выделенном Сервере Ubuntu 18.04  

### Установка Asterisk
Asterisk это бесплатный opensource (с открытым исходным кодом) продукт организации IP-телефонии. Он появился в 2004 году и это была революцию на рынке телефонии. На тот момент если тебе нужно было организовать связь в офисе из 10 сотрудников ты должен был купить Avaya, Panasonic или Samsung минимум за $2000. Asterisk полностью бесплатный.  

Настройка Asterisk непростое дело. Путь звонка (в астере он называется дИалплан) описывается в конфигурационном файле и немного похож на программирование.
Для него сделали веб-интерфейс FreePBX, который тоже не самое простое что есть на свете!  

Нам нужно установить Asterisk (сервер телефонии) , FreePBX (веб-интерфейс к нему), базу данных в которой будут храниться CDR ( статистика звонков)   

Рекомендую не смешивать телефонию с другими сервисами. Телефония работает в реальном времени. Ты можешь подождать пока загрузится сайт, но врядли будешь доволен когда твой голос дойдет до собеседника с задержкой. **Всегда используй для телефонии отдельный сервер или отдельную виртуальную машину**.  

Установим пакеты, от которых зависит Asterisk. В тоже время сам Asteirsk мы будем ставить из исходных кодов. Есть готовый дистибутив **AsteriskNOW**, **НО нам для обучения** будет полезно собрать все самим.  

Скачаем дистрибутив Asterisk. Последняя стабильная версия нам не подходит (Дата статьи: 25.05.2020). FreePBX с ней пока не умеет работать. Поэтому ищем архивные и скачиваем 13-ю версию.

```wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-13-current.tar.gz```

Затем распакуем скачанный архив:  

```sudo tar xzfv asterisk-13-current.tar.gz```

Так теперь мы запустим скрипт configure, который проверит нашу ОС, чтобы понять все ли есть для сборки Asterisk. "Сборка" - это превращение или компиляция текстов программ в исполняемые файлы - те что можно запускать. Этот процесс делает специальная программа.  

Теперь запустим сборку зависимостей:  

```./configure```

***./*** **- это указание что нам нужна команда configure, которая находится в том же каталоге что и находимся мы. Если ввести просто configure, тогда Linux попробует запустить команду configure, которая находится в каталогах с программами, указанными в переменной PATH.**  

По итогу могут вылазить ошибки. Связано это с отсутствием компилятора. Решение:

```sudo apt-get install build-essential```  
```sudo apt-get install libedit-dev```  

Далее Asterisk подсказывает какие ему нужны доп.библиотеки для компиляции. Необходимо их установить. Сборка считается завершенной, если в командной строке появится товарный знак Asterisk!  


Теперь воспользуемся рекомендацией из официальной документации Asterisk (внизу страницы).  

https://wiki.asterisk.org/wiki/display/AST/Checking+Asterisk+Requirements  

Там написано что можно использовать скрипт который установит пакеты, которых не хватает.  

```sudo ./install_prereq install```  
```sudo ./install_prereq install-unpackaged```  

```sudo ./configure --with-pjproject-bundled```  

Вроде все ок. Все зависимости учтены. Давай запустим компиляцию. Напомню - это когда из исходных текстов программа превращяется в бинарный файл, который можно запустить.

```sudo make```

make install раскидает файлы по папкам. Установка готового ПО.  

```sudo make install```  

Создадим конфиги по-умолчанию и запустим asterisk:

```sudo make samples```  
```sudo make congig```  
```sudo systemctl start asterisk```

Проверим запуск:

```sudo asterisk -r```  
### Установка FreeBPX  

Теперь давай займемся установкой веб-интерфейса для Asterisk - FreePBX. FreePBX дает тоже готовый Live CD (готовый дистрибутив), но в целях обучения нам это не нужно.  
Идем на страницу скачивания https://www.freepbx.org/downloads/freepbx-distro/. У кнопки скачивания копируем ссылку и дальше качаем дистрибутив.

Установка по оф.документации.  
https://wiki.freepbx.org/display/FOP/Installing+FreePBX+14+on+Ubuntu+18.04#InstallingFreePBX14onUbuntu18.04-Loginas,orswitchto,theRootUser  

Установка Коннектора ODBC:  
```wget https://dev.mysql.com/get/Downloads/Connector-ODBC/5.3/mysql-connector-odbc-5.3.14-linux-ubuntu18.04-x86-64bit.tar.gz```  
```tar -xzvf mysql-connector-odbc-5.3.14-linux-ubuntu18.04-x86-64bit.tar.gz```  
```mkdir /usr/lib/odbc```  
```cp mysql-connector-odbc-5.3.14-linux-ubuntu18.04-x86-64bit/lib/* /usr/lib/odbc/```

Дальше по инструкции!  

FreePBX имеет кучу модулей, расширяющих базовый функционал.  
```sudo fwconsole ma downloadinstall manager```  
```sudo fwconsole ma downloadinstall asteriskinfo```  
```sudo fwconsole ma downloadinstall queues```  

Модуль **queues** позволит настраивать входящую очередь звонков.  
Модуль **asteriskinfo** даст нам возможность отладки настроек в графическом режиме.  Чтобы не залезать на сервер по ssh и не заходить в asterisk -r.  
От модуля **arimanager** зависит модуль asteriskinfo и поэтому устанавливается перед ним.  

Установщик к сожалению не ставит сервис в автозагрузку. Для этого нам придется еще немного потрудиться. В качестве системы инициализации в debian и ubuntu трудится systemd. Мы напишем под него конфигурацию сервиса freepbx.  

В mc или nano открой файл **/etc/systemd/system/freepbx.service** и добавить:    

*[Unit]*  
*Description=FreePBX VoIP Server*  
*After=mariadb.service*  

*[Service]*  
*Type=oneshot*  
*RemainAfterExit=yes*  
*ExecStart=/usr/sbin/fwconsole start -q*  
*ExecStop=/usr/sbin/fwconsole stop -q*  

*[Install]*  
*WantedBy=multi-user.target*  

После этого мы можем установить сервис FreePBX в автозагрузку:  
```systemctl enable freepbx.service```  
```systemctl start freepbx.service```  

















