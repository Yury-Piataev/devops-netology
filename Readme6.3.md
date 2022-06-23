# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

### Ответ:
``` 
# docker run --rm --name mysql -e MYSQL_ROOT_PASSWORD=mysql -it -p 3306:3306 -v vol1:/etc/mysql/data -d mysql:8.0

cp /home/vir/1/test_dump.sql ./var/lib/docker/volumes/vol1/_data

mysql -u root -p

CREATE DATABASE test_db DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
   
mysql -u root -p test_db < /etc/mysql/data/test_dump.sql

mysql> \s
--------------
mysql  Ver 8.0.29 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:        17
Current database:    test_db
Current user:        root@localhost
SSL:            Not in use
Current pager:        stdout
Using outfile:        ''
Using delimiter:    ;
Server version:        8.0.29 MySQL Community Server - GPL
Protocol version:    10
Connection:        Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb3
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:        /var/run/mysqld/mysqld.sock
Binary data as:        Hexadecimal
Uptime:            54 min 11 sec

Threads: 2  Questions: 122  Slow queries: 0  Opens: 152  Flush tables: 3  Open tables: 70  Queries per second avg: 0.037
--------------

mysql> show tables
    -> ;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)


mysql> SELECT COUNT(*) FROM orders WHERE price > 300;
+----------+
| COUNT(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

 ```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней
- количество попыток авторизации - 3
- максимальное количество запросов в час - 100
- атрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привилегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и
**приведите в ответе к задаче**.
### Ответ:
```
mysql> CREATE USER 'test' IDENTIFIED BY 'test-pass'
    ->     WITH
    ->     MAX_QUERIES_PER_HOUR 100
    ->     PASSWORD EXPIRE INTERVAL 180 DAY
    ->     FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2
    ->     ATTRIBUTE '{"FirstName":"James", "LastName":"Pretty"}';

mysql> GRANT SELECT ON test_db.orders TO 'test';

mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER='test';
+------+------+----------------------------------------------+
| USER | HOST | ATTRIBUTE                                    |
+------+------+----------------------------------------------+
| test | %    | {"LastName": "Pretty", "FirstName": "James"} |
+------+------+----------------------------------------------+
1 row in set (0.01 sec)

```
## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`
### Ответ:
```
mysql> SELECT TABLE_NAME,ENGINE FROM information_schema.TABLES WHERE table_name = 'orders' and TABLE_SCHEMA = 'test_db';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | InnoDB |
+------------+--------+
1 row in set (0.00 sec)

mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.04 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                                                                                                |
+----------+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
|        1 | 0.00361925 | SELECT TABLE_NAME,ENGINE,ROW_FORMAT,TABLE_ROWS,DATA_LENGTH,INDEX_LENGTH FROM information_schema.TABLES WHERE table_name = 'orders' and  TABLE_SCHEMA = 'test_db' ORDER BY ENGINE asc |
|        2 | 0.00017775 | SET profiling = 1                                                                                                                                                                    |
|        3 | 0.02494425 | ALTER TABLE orders ENGINE = MyISAM                                                                                                                                                   |
|        4 | 0.02761975 | ALTER TABLE orders ENGINE = InnoDB                                                                                                                                                   |
+----------+------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```
## Задача 4

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буфера с незакомиченными транзакциями 1 Мб
- Буфер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

### Ответ:
```

* root@ae1ba9d0d7c4:/# echo 'innodb_log_buffer_size = 1M' >> /etc/mysql/my.cnf
* root@ae1ba9d0d7c4:/# echo 'key_buffer_size = 640М'>> /etc/mysql/my.cnf
* root@ae1ba9d0d7c4:/# echo 'innodb_flush_log_at_trx_commit = 0'  >> /etc/mysql/my.cnf
* root@ae1ba9d0d7c4:/# echo 'innodb_file_format=Barracuda' >> /etc/mysql/my.cnf
* root@ae1ba9d0d7c4:/# echo 'max_binlog_size = 100M' >> /etc/mysql/my.cnf

root@ae1ba9d0d7c4:/# cat /etc/mysql/my.cnf
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html

[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/
innodb_log_buffer_size = 1M
key_buffer_size = 640М
innodb_flush_log_at_trx_commit = 0
innodb_file_format=Barracuda
max_binlog_size = 100M
```
