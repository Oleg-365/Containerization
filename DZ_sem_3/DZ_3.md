Задание:
1) запустить контейнер с БД, отличной от mariaDB, используя инструкции на сайте: https://hub.docker.com/
2) добавить в контейнер hostname такой же, как hostname системы через переменную
3) заполнить БД данными через консоль
4) запустить phpmyadmin (в контейнере) и через веб проверить, что все введенные данные доступны

Формат сдачи ДЗ: предоставить доказательства выполнения задания посредством ссылки на google-документ с правами на комментирование/редактирование.
Результатом работы будет: текст объяснения, логи выполнения, история команд и скриншоты (важно придерживаться такой последовательности).
В названии работы должны быть указаны ФИ, номер группы и номер урока.

ДЗ_семинара_3
хост-система.

# docker run --rm -d \
создать контейнер
--rm		-- опция, удаляющая контейнер автоматически после остановки, несовместима с правилами перезагрузки, совместима с режимом -d (согласно man-page).
-d		-- "Detached mode", опция запускает контейнер в фоновом режиме.
> --name hw03_mariadb \
опция задаёт имя контейнера.
> -v /home/HomeWork03/mdb_testdb:/var/lib/mysql \
опция указывает директорию (абсолютный путь, указывается до ':') которую необходимо смонтировать в контейнер с путём, указываемым после ':' (дополнительно можно указать опции монтирования, в частности rw/ro)
> -e MARIADB_ROOT_PASSWORD=dwp_toor \
переменная окружения в контейнере, пароль суперпользователя для БД
> -e MARIADB_USER=ordinary_user \
переменная окружения в контейнере, имя пользователя для БД
> -e MARIADB_PASSWORD=dwp_resu \
переменная окружения в контейнере, пароль пользователя для БД
> mariadb:10.10.2
имя образа контейнера

8aa4983aa90ef5e3646fd57f5d22b82f2e0bc6e487945b254e2c18db26fe1dab
вывод команды.

# docker ps -a
посмотреть запущенные контейнеры

CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS      NAMES
8aa4983aa90e   mariadb:10.10.2   "docker-entrypoint.s…"   17 seconds ago   Up 14 seconds   3306/tcp   hw03_mariadb

# docker logs hw03_mariadb 
посмотреть логи контейнера.

2023-02-04 12:20:55+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.10.2+maria~ubu2204 started.
2023-02-04 12:20:56+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2023-02-04 12:20:56+00:00 [Note] [Entrypoint]: Entrypoint script for MariaDB Server 1:10.10.2+maria~ubu2204 started.
2023-02-04 12:20:56+00:00 [Note] [Entrypoint]: Initializing database files


команда:
# docker exec -it hw03_mariadb bash
выполнить команду в запущенном контейнере
-i		-- в интерактивном режиме
-t		-- подключить псевдо-терминал
bash		-- команда для запуска

контейнер.

команда:
root@8aa4983aa90e:/# mariadb -u ordinary_user -p
подключиться к БД как пользователь.

MariaDB [(none)]> create database hw03_test_base;
ERROR 1044 (42000): Access denied for user 'ordinary_user'@'%' to database 'hw03_test_base'
MariaDB [(none)]> exit
Bye

root@8aa4983aa90e:/# mariadb -u root -p
подключиться к БД как суперпользователь, чтобы раздать права.


MariaDB [(none)]> create database hw03_test_base;
Query OK, 1 row affected (0.000 sec)
создана БД для ДЗ.

MariaDB [mysql]> GRANT ALL PRIVILEGES ON  *.* to 'ordinary_user';
Query OK, 0 rows affected (0.066 sec)
выданы обширные права пользователю.

MariaDB [(none)]> select * from mysql.user where User='ordinary_user'\G
посмотреть что права выданы (листинг вывода пропущен).

MariaDB [(none)]> \q
Bye

root@8aa4983aa90e:/# mariadb -u ordinary_user -p
подключиться к БД пользователем.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| hw03_test_base     |		// созданная под рутом БД
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.000 sec)


MariaDB [(none)]> use hw03_test_base;
Database changed
переаключиться на БД.


Ниже добавление таблиц в БД для дальнейшей проверки.
MariaDB [hw03_test_base]> create table tst_tbl01(col1 int, col2 varchar(20));
Query OK, 0 rows affected (0.405 sec)

MariaDB [hw03_test_base]> insert tst_tbl01(col1, col2) values(1, 'one'), (2, 'two'), (3, 'three');
Query OK, 3 rows affected (0.052 sec)
Records: 3  Duplicates: 0  Warnings: 0

MariaDB [hw03_test_base]> create table tst_tbl02(col3 int, col4 varchar(20));
Query OK, 0 rows affected (0.299 sec)

MariaDB [hw03_test_base]> insert tst_tbl02(col3, col4) values(4, 'four'), (5, 'five'), (6, 'six'), (, );

MariaDB [hw03_test_base]> select * from tst_tbl01;
+------+-------+
| col1 | col2  |
+------+-------+
|    1 | one   |
|    2 | two   |
|    3 | three |
+------+-------+
3 rows in set (0.000 sec)

MariaDB [hw03_test_base]> select * from tst_tbl02;
+------+------+
| col3 | col4 |
+------+------+
|    4 | four |
|    5 | five |
|    6 | six  |
|    7 | NULL |
+------+------+
4 rows in set (0.000 sec)

хост-система.
# ls HomeWork03/mdb_testdb/hw03_test_base/ -lah
total 148K
просмотр содержимого ранее примонтированной к контейнеру директории.

# docker stop hw03_mariadb 
hw03_mariadb

# docker --rm hw03_mariadb

# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

Далее следует повторное создание контейнера той же командой, что и раньше. При подключении БД должна быть доступна и работать.

Вместе с этим, необходимо вывести скриншот, подтверждающий то, что есть подключение к БД из другого контейнера:
 
