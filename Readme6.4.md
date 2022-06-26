# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

### Ответ:
```
docker run --rm --name postgres -e POSTGRES_PASSWORD=postgres -it -p 5432:5432 -v /home/vir/1/psql:/etc/postgres  -d postgres:13

psql -Upostgres - подключение к базе.
\l - вывод списка баз
\c <database>  - подключение к базе с именем < >
\dt *.* - вывод списка таблиц
\d - описание содержимого таблиц
\q  - выход из psql
```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders`
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.

### Ответ:
```
postgres=# CREATE DATABASE test_database;
root@d87ad8be5e38:/# psql test_database -Upostgres < /etc/postgres/test_dump.sql
root@d87ad8be5e38:/# psql -U postgres
postgres=# \c test_database
test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE

est_database=# select avg_width from pg_stats where tablename='orders';
 avg_width
-----------
         4
        16
         4
(3 rows)
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

### Ответ:
```
test_database=# CREATE TABLE orders_1 (LIKE orders);
CREATE TABLE
test_database=# CREATE TABLE orders_2 (LIKE orders);
CREATE TABLE
test_database=# INSERT INTO orders_1 SELECT * FROM orders WHERE price>499;
INSERT 0 3
test_database=# INSERT INTO orders_2 SELECT * FROM orders WHERE price<=499;
INSERT 0 5
test_database=# DROP TABLE orders;
DROP TABLE
test_database=#

Можно было избежать ручного разбиения, если при проектировании сделать таблицу секционированной.
```

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

### Ответ:
```
root@d87ad8be5e38:/# pg_dump -U postgres -d test_database > /etc/postgres/test_database_dump.sql


Для уникальности можно добавить индекс, ключ или добавить параметр UNIQUE.

```

---
