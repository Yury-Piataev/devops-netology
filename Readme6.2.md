# Домашнее задание к занятию "6.2. SQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

### Ответ:
```
root@vir-PC:~# docker volume create vol1
root@vir-PC:~# docker volume create vol2
root@vir-PC:~# docker run --rm --name postgres -e POSTGRES_PASSWORD=postgres -it -p 5432:5432 -v vol1:/var/lib/postgresql/data -v vol2:/var/lib/postgresql -d postgres:12
```
## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

### Ответ:

```
psql -U postgres
CREATE DATABASE “test_db”;

CREATE ROLE "test-admin-user" PASSWORD 'password' SUPERUSER NOCREATEDB NOCREATEROLE INHERIT LOGIN;

CREATE TABLE orders ( 
        id serial PRIMARY KEY, 
        name text, 
        price integer
    );

CREATE TABLE clients (
        id serial PRIMARY KEY, 
        surname text,
        country text,
        zakaz integer,
        FOREIGN KEY (zakaz) REFERENCES orders (id)
    );

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO "test-admin-user";

CREATE ROLE "test-simple-user" with encrypted password 'password';
    
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO "test-simple-user";

postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 “test_db” | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)

postgres=# \du
                                       List of roles
    Role name     |                         Attributes                         | Member of
------------------+------------------------------------------------------------+-----------
 postgres         | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test-admin-user  | Superuser                                                  | {}
 test-simple-user | Cannot login                                               | {}

postgres=# SELECT grantee, privilege_type
    FROM information_schema.role_table_grants
    WHERE table_name='clients';
     grantee      | privilege_type
------------------+----------------
 postgres         | INSERT
 postgres         | SELECT
 postgres         | UPDATE
 postgres         | DELETE
 postgres         | TRUNCATE
 postgres         | REFERENCES
 postgres         | TRIGGER
 test-admin-user  | INSERT
 test-admin-user  | SELECT
 test-admin-user  | UPDATE
 test-admin-user  | DELETE
 test-admin-user  | TRUNCATE
 test-admin-user  | REFERENCES
 test-admin-user  | TRIGGER
 test-simple-user | INSERT
 test-simple-user | SELECT
 test-simple-user | UPDATE
 test-simple-user | DELETE
(18 rows)
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

### Ответ:
```
INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);

INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), 
(4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');

SELECT COUNT (*) FROM orders;
select count (*) from clients;

postgres=# SELECT COUNT (*) FROM orders;
 count 
-------
     5
(1 row)

postgres=# select count (*) from clients;
 count 
-------
     5
(1 row)
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.

### Ответ:
```
UPDATE clients SET zakaz = 3 WHERE id = 1;UPDATE clients SET zakaz = 4 WHERE id = 2;UPDATE clients SET zakaz = 5 WHERE id = 3;

SELECT * FROM clients WHERE zakaz IS NOT NULL;

postgres=# SELECT * FROM clients WHERE zakaz IS NOT NULL;
 id |       surname        | country | zakaz 
----+----------------------+---------+-------
  1 | Иванов Иван Иванович | USA     |     3
  2 | Петров Петр Петрович | Canada  |     4
  3 | Иоганн Себастьян Бах | Japan   |     5
(3 rows)
```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

### Ответ:
```
EXPLAIN SELECT * FROM clients WHERE zakaz IS NOT NULL;

postgres=# EXPLAIN SELECT * FROM clients WHERE zakaz IS NOT NULL;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: (zakaz IS NOT NULL)
(2 rows)

```
EXPLAIN выводит информацию о плане выполнения конкретного запроса. Чтение данных из таблицы может выполняться несколькими способами. 
Seq Scan — последовательное, блок за блоком, чтение данных таблицы. Параметр cost(затраты) — это оценки времени, которое, как ожидается, 
займет узел. По умолчанию затраты указаны в единицах времени последовательного чтения блока размером 8 КБ. Каждый узел имеет две затраты: 
стартовую стоимость и общую стоимость. Rows - редполагаемое количество строк, width предполагаемая ширина каждой строки. 
Планировщик выбирает план на основе самого дешевого варианта.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

### Ответ:
```
Создание бэкапа:
pg_dump -Upostgres test_db > /var/lib/postgresql/220622.dump

В новом контейнере нужно создать пустую базу:
CREATE DATABASE test_dbn;

и восстановить данные:
root@e7cbb22744ea:/# psql -Upostgres -d test_dbn < /var/lib/postgresql/220622.dump
```
---
