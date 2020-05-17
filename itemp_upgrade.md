## Как обновлять сайт itempuniversity.com

### Шаг 1. Создание сервера.
Зайдите на сайт https://aws.amazon.com/ и создайте сервер с такими характеристиками:
- ОС: Ubuntu bionic (20.04)
- Регион: Frankfurt
- Instance: t2.micro
- Название: itemp-3.8 (3.8 - это версия moodle, укажите соответствующую)

### Шаг 2. Установка пакетов.
Зайдите на новый сервер. Выполните команда:
`sudo su -`
затем команды:
```
apt update && apt install -y lftp git nginx php-fpm php-xml php-curl php-mysql \
php-zip php-intl php-mbstring php-xmlrpc php-gd php-soap \
software-properties-common certbot graphviz ghostscript aspell
 ```
 
### Шаг 3. Скачивание новой версии moodle.
Возьмите название ветки нового moodle здесь: https://github.com/moodle/moodle/
Например, версии 3.8 соответствует ветка MOODLE_38_STABLE
Затем зайдите на новый сервер. Выполните команду:
```
cd /var/www && git clone https://github.com/moodle/moodle.git -b MOODLE_38_STABLE
```

### Шаг 4. Перевод сайта в режим обслуживания.

### Шаг 5. Создание бэкапа БД.
Делаем бэкап на aws.amazon.com в RDS.

### Шаг 6. Перенос файлов (плагинов и т.п.) со старого сервера.

Зайдите на старый сервер и выполните команду `sudo su -`, затем `ssh-keygen`. Затем пять раз нажмите Enter.
Выполните команду `cat /root/.ssh/id_rsa.pub`. Скопируйте вывод этой команды.
Зайдите на новый сервер, и добавьте скопированный ключ в конец файла `/root/.ssh/authorized_keys`

Вновь зайдите на старый сервер, и смотрим список измененных файлов с помощью команды:
`cd /var/www/moodle && git status`
затем  выполните команды (`54.93.99.81` - адрес нового сервера. 
В вашем случае адрес будет другим.):

```
scp /var/www/moodle/config.php  root@54.93.99.81:/var/www/moodle/

scp /etc/nginx/conf.d/itemp.conf \
root@54.93.99.81:/etc/nginx/conf.d/itemp.conf

scp -r /var/www/moodle/images \
root@54.93.99.81:/var/www/moodle/

scp -r /var/www/moodle/mod/customcert/ \
root@54.93.99.81:/var/www/moodle/mod

scp -r /var/www/moodle/enrol/autoenrol/ \
root@54.93.99.81:/var/www/moodle/enrol/

scp -r /etc/letsencrypt/ \
root@54.93.99.81:/etc/

scp -r /var/moodledata/ root@54.93.99.81:/var/
```


### Шаг 7. Изменение прав на каталоги. Рестарт NGINX.
На новом сервере:
```
rm -f /var/www/moodle/install.php
chown -R www-data. /var/moodledata
chown -R www-data. /var/www/moodle/
service nginx reload
```

### Шаг 8.
На новом сервере:
Изменить параметр $CFG->dbhost на новый инстанс RDS.

Изменить версию php при необходимости а файле /etc/nginx/conf.d/itemp.conf (2 строка)

`service nginx reload`

### Шаг 9. Изменение /etc/hosts и обновление moodle.
На своем компьютере выполните команду: `sudo bash -c 'echo "54.93.99.81 itempuniversity.com" >> /etc/hosts'` (у вас ip адрес будет другой!)
После этого ваш компьютер при входе через браузер на сайт itempuniversity.com будет заходить на новый сервер.

Зайдите на itempuniversity.com через браузер и обновите сайт.

### Шаг 10. Проверка работоспособности.
Проверьте через браузер, что все функции сайта работают. Залогиньтесь. Покликайте по ссылкам на курсы и т. п.

### Шаг 11. Изменение ip адреса сервера.

Переносим Elastic IP address на новый сервер.

### Шаг 12. Вывод сайта из режима обслуживания.

### Шаг 13. Копирование crontab.

На новом сервере:
`crontab -e `
и вставить строку:

```
* * * * * /usr/bin/php  /var/www/moodle/admin/cli/cron.php >/dev/null
```

### Шаг 14. Очистка /etc/hosts
На своем компьютере удалите сроку с сайтом itempuniversity.com из файла /etc/hosts
Проверьте через браузер, что сайт itempuniversity.com работает.

### Шаг 15. Изменение настроек php
На новом сервере открыть файл `/etc/php/7.4/fpm/php.ini` и установить следующие параметры:
```
max_execution_time = 900
post_max_size = 1000M
upload_max_filesize = 1000M
```
Затем перезапустить сервис:
```
systemctl reload php7.4-fpm
```

### Шаг 16. Удаление старого сервера
Если в течении нескольких дней с момента обновления не было никаких жалоб на обновленный сайт, то удаляем старый сервер.
Удаляем старую базу данных RDS после удаления сервера.

