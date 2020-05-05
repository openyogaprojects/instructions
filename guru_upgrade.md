## Как обновлять сайт guru.openyogaclass.com

### Шаг 1. Создание сервера.
Зайдите на сайт https://console.scaleway.com/ и создайте сервер с такими характеристиками:
- ОС: Ubuntu bionic (18.04)
- Регион: Амстердам
- Instance: Development/DEV-1S
- Название: guru-3.8 (3.8 - это версия moodle, укажите соответствующую)

### Шаг 2. Установка пакетов.
Зайдите на новый сервер. Выполните команды:

```
add-apt-repository ppa:certbot/certbot
```

```
apt update && apt install -y lftp git nginx php-fpm php-xml php-curl \
php-mysql php-zip php-intl php-mbstring php-xmlrpc php-gd php-soap \ 
mysql-server software-properties-common certbot python-certbot-nginx \
graphviz ghostscript aspell
 ```
 
### Шаг 3. Скачивание новой версии moodle.
Возьмите название ветки нового moodle здесь: https://github.com/moodle/moodle/
Например, версии 3.8 соответствует ветка MOODLE_38_STABLE
Затем зайдите на новый сервер. Выполните команду:
```
cd /var/www && git clone https://github.com/moodle/moodle.git -b MOODLE_38_STABLE
```

### Шаг 4. Перенос файлов (плагинов и т.п.) со старого сервера.

Зайдите на старый сервер и выполните команду `ssh-keygen`. Затем пять раз нажмите Enter.
Выполните команду `cat /root/.ssh/id_rsa.pub`. Скопируйте вывод этой команды.
Зайдите на новый сервер, и добавьте скопированный ключ в конец файла `/root/.ssh/authorized_keys`

Вновь зайдите на старый сервер и выполните команды (`51.15.122.178` - адрес нового сервера. 
В вашем случае адрес будет другим.):

```
scp /etc/nginx/sites-available/guru.openyogaclass.com.conf \
root@51.15.122.178:/etc/nginx/sites-available/guru.openyogaclass.com.conf
scp /var/www/moodle/theme/classic/pix/favicon.ico \
root@51.15.122.178:/var/www/moodle/theme/classic/pix/favicon.ico
scp /var/www/moodle/privacypolicy.htm \
root@51.15.122.178:/var/www/moodle/privacypolicy.htm
scp /var/www/moodle/config.php \
root@51.15.122.178:/var/www/moodle/config.php
scp /opt/upload_backups.sh  root@51.15.122.178:/opt/
scp /root/my.cnf  root@51.15.122.178:/root/
scp -r /var/www/moodle/lib/tcpdf/config/lang/ \
root@51.15.122.178:/var/www/moodle/lib/tcpdf/config/lang/
scp -r /var/www/moodle/mod/certificate/ \
root@51.15.122.178:/var/www/moodle/mod/certificate/
scp -r /var/www/moodle/mod/checklist/ \
root@51.15.122.178:/var/www/moodle/mod/checklist/
scp -r /var/www/moodle/mod/customcert/ \
root@51.15.122.178:/var/www/moodle/mod/customcert/
scp -r /etc/letsencrypt/ \
root@51.15.122.178:/etc/letsencrypt/
```

### Шаг 5. Установка Mysql.
Зайдите на новый сервер. Выполните команду:
```
mysql_secure_installation
```

Затем ответьте на вопросы:

Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: **N**

New password: **придумайте пароль**

Remove anonymous users? (Press y|Y for Yes, any other key for No) : **Y**

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : **Y**

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : **Y**

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : **Y**

### Шаг 6. Перевод сайта в режим обслуживания.

### Шаг 7. Создание бэкапа БД и копирование moodledata.
На старом сервере:
```
mysqldump -p -u moodle moodle > moodle.sql
scp moodle.sql  root@51.15.122.178:/
scp -r /var/moodledata/ root@51.15.122.178:/var/moodledata/
```

### Шаг 8. Создание БД на новом севрере и развертывание из бэкапа.
На новом сервере:
```
mysql -u root -p
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'moodle'@'localhost' IDENTIFIED BY 'Absolut777';
GRANT ALL PRIVILEGES ON moodle.* TO 'moodle'@'localhost' IDENTIFIED BY 'Absolut777';
FLUSH PRIVILEGES;
exit
mysql -u root -p moodle < /moodle.sql
```

### Шаг 8. Изменение прав на каталоги. Рестарт NGINX.
На новом сервере:
```
chown -R www-data. /var/moodledata
ln -s /etc/nginx/sites-available/guru.openyogaclass.com.conf /etc/nginx/sites-enabled/
rm -f /var/www/moodle/install.php
service nginx restart
```

### Шаг 9. Изменение /etc/hosts и обновление moodle.
На своем компьютере выполните команду: `sudo bash -c 'echo "51.15.122.178 guru.openyogaclass.com" >> /etc/hosts'` (у вас ip адрес будет другой!)
После этого ваш компьютер при входе через браузер на сайт guru.openyogaclass.com будет заходить на новый сервер.

Зайдите на guru.openyogaclass.com через браузер и обновите сайт.

### Шаг 10. Проверка работоспособности.
Проверьте через браузер, что все функции сайта работают. Залогиньтесь. Покликайте по ссылкам на курсы и т. п.

### Шаг 11. Изменение TXT и A записи DNS

11.1 Зайти на openyogaclass.com/cpanel через браузер

11.2 Кликнуть по пункту "Advanced Zone Editor"

11.3 В открывшейся вкладке выберите домен openyogaclass.com

11.4 В открывшейся вкладке найдите А-записи guru.openyogaclass.com. и www.guru.openyogaclass.com. и измените ip адрес на новый

11.5 Найдите TXT-запись, в которой присутствует строка spf.google.com, и добавьте туда ip адрес нового сервера.
    Примерно, это должно выглядеть так:
    
    Было:
    v=spf1 ptr mx ip4:198.57.247.184 ip4:51.15.47.41 ip4:51.15.40.248 include:_spf.google.com ~all
    
    Стало:
    v=spf1 ptr mx ip4:198.57.247.184 ip4:51.15.47.41 ip4:51.15.40.248 ip4:51.15.122.178 include:_spf.google.com ~all

### Шаг 12. Вывод сайта из режима обслуживания.

### Шаг 13. Копирование crontab.

На старом сервере выполнить команду:
`crontab -l`
и скопировать вывод команды.

На новом сервере:
`crontab -e `
и вставить то что скопировали.

Затем на новом сервере:
```
mkdir -p /root/gurubackup/databases
mkdir -p /backups/guru_openyogaclass_com/
```

### Шаг 14. Удаление crontab на старом сервере.
На старом сервере выполнить команду:
`crontab -e `
и удалить все содержимое.

### Шаг 15. Очистка /etc/hosts
На своем компьютере удалите сроку с сайтом guru.openyogaclass.com из файла /etc/hosts
Проверьте через браузер, что сайт guru.openyogaclass.com работает.

### Шаг 16. Изменение настроек php
На новом сервере открыть файл `/etc/php/7.2/fpm/php.ini` и установить следующие параметры:
```
max_execution_time = 900
post_max_size = 1000M
upload_max_filesize = 1000M
```
Затем перезапустить сервис:
```
systemctl reload php7.2-fpm
```

### Шаг 17. Удаление старого сервера
Если в течении недели с момента обновления не было никаких жалоб на обновленный сайт, то удаляем старый сервер.






