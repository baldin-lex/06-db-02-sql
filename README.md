# Домашнее задание к занятию 2. «SQL» - Балдин

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

## Решение:

*Содержимое docker-compose.yaml*
```yaml
version: '3'

services:

  postgres:
    image: postgres:12
    container_name: postgres
    ports:
      - "0.0.0.0:5432:5432"
    volumes:
      - data:/var/lib/postgresql/data
      - backup:/media/postgresql/backup
    environment:
      POSTGRES_USER: "baldin"
      POSTGRES_PASSWORD: "qwerty"
      POSTGRES_DB: "test_db"
    restart: always

volumes:
  data:
  backup:
```
*Запускаю:*
```bash
[admin@hw-06-02-docker ~]$ docker-compose up -d
Creating network "admin_default" with the default driver
Creating volume "admin_database_volume" with default driver
Creating volume "admin_backup_volume" with default driver
Pulling db (postgres:12)...
12: Pulling from library/postgres
f03b40093957: Pull complete
9d674c93414d: Pull complete
de781e8e259a: Pull complete
5ea6efaf51f6: Pull complete
b078d5f4ac82: Pull complete
97f84fb2a918: Pull complete
5a6bf2f43fb8: Pull complete
f1a40e88fea4: Pull complete
ea9811dc38ae: Pull complete
515634916e47: Pull complete
81b43500849b: Pull complete
800a983fb77c: Pull complete
3ecd088e33c5: Pull complete
Digest: sha256:7db33237a29afa0a62998b7b6707bfe99766e9da02b5780f8db8f773b85c8f33
Status: Downloaded newer image for postgres:12
Creating postgres ... done
```
*Иду в контейнер:*
```bash
[admin@hw-06-02-docker ~]$ sudo docker exec -it postgres bash
root@f5b11e40a0c6:/# 
```
*Подключаюсь к базе:*
```bash
baldin@42a9c452187f:/$ psql test_db -U baldin
psql (12.15 (Debian 12.15-1.pgdg110+1))
Type "help" for help.

test_db=# 
```

---

## Задача 2

В БД из задачи 1: 

- создайте пользователя test-admin-user и БД test_db;
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже);
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db;
- создайте пользователя test-simple-user;
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE этих таблиц БД test_db.

Таблица orders:

- id (serial primary key);
- наименование (string);
- цена (integer).

Таблица clients:

- id (serial primary key);
- фамилия (string);
- страна проживания (string, index);
- заказ (foreign key orders).

Приведите:

- итоговый список БД после выполнения пунктов выше;
```bash
test_db=# \l
                                   List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    |      Access privileges      
-----------+--------+----------+------------+------------+-----------------------------
 postgres  | baldin | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | baldin | UTF8     | en_US.utf8 | en_US.utf8 | =c/baldin                  +
           |        |          |            |            | baldin=CTc/baldin
 template1 | baldin | UTF8     | en_US.utf8 | en_US.utf8 | =c/baldin                  +
           |        |          |            |            | baldin=CTc/baldin
 test_db   | baldin | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/baldin                 +
           |        |          |            |            | baldin=CTc/baldin          +
           |        |          |            |            | "test-simple-user"=c/baldin
(4 rows)
```
- описание таблиц (describe);
```bash
test_db=# \d clients
                                       Table "public.clients"
      Column       |       Type        | Collation | Nullable |               Default               
-------------------+-------------------+-----------+----------+-------------------------------------
 id                | integer           |           | not null | nextval('clients_id_seq'::regclass)
 фамилия           | character varying |           |          | 
 страна проживания | character varying |           |          | 
 заказ             | integer           |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
```
```bash
test_db=# \d orders
                                    Table "public.orders"
    Column    |       Type        | Collation | Nullable |              Default               
--------------+-------------------+-----------+----------+------------------------------------
 id           | integer           |           | not null | nextval('orders_id_seq'::regclass)
 наименование | character varying |           |          | 
 цена         | integer           |           |          | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.
```bash
test_db=# SELECT grantee, table_name, privilege_type FROM information_schema.table_privileges WHERE table_name IN ('orders','clients');
     grantee      | table_name | privilege_type 
------------------+------------+----------------
 baldin           | orders     | INSERT
 baldin           | orders     | SELECT
 baldin           | orders     | UPDATE
 baldin           | orders     | DELETE
 baldin           | orders     | TRUNCATE
 baldin           | orders     | REFERENCES
 baldin           | orders     | TRIGGER
 test-admin-user  | orders     | INSERT
 test-admin-user  | orders     | SELECT
 test-admin-user  | orders     | UPDATE
 test-admin-user  | orders     | DELETE
 test-admin-user  | orders     | TRUNCATE
 test-admin-user  | orders     | REFERENCES
 test-admin-user  | orders     | TRIGGER
 test-simple-user | orders     | INSERT
 test-simple-user | orders     | SELECT
 test-simple-user | orders     | UPDATE
 test-simple-user | orders     | DELETE
 baldin           | clients    | INSERT
 baldin           | clients    | SELECT
 baldin           | clients    | UPDATE
 baldin           | clients    | DELETE
 baldin           | clients    | TRUNCATE
 baldin           | clients    | REFERENCES
 baldin           | clients    | TRIGGER
 test-admin-user  | clients    | INSERT
 test-admin-user  | clients    | SELECT
 test-admin-user  | clients    | UPDATE
 test-admin-user  | clients    | DELETE
 test-admin-user  | clients    | TRUNCATE
 test-admin-user  | clients    | REFERENCES
 test-admin-user  | clients    | TRIGGER
 test-simple-user | clients    | INSERT
 test-simple-user | clients    | SELECT
 test-simple-user | clients    | UPDATE
 test-simple-user | clients    | DELETE
(36 rows)
```
---

## Задача 3

Используя SQL-синтаксис, наполните таблицы следующими тестовыми данными:

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

Используя SQL-синтаксис:
- вычислите количество записей для каждой таблицы.

Приведите в ответе:
    
    - запросы,
    - результаты их выполнения.
    
## Решение:

```bash
test_db=# INSERT INTO orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
test_db=# INSERT INTO clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
test_db=# SELECT count(1) FROM orders;
 count 
-------
     5
(1 row)

test_db=# SELECT count(1) FROM clients;
 count 
-------
     5
(1 row)

test_db=# SELECT * FROM orders;
 id | наименование | цена 
----+--------------+------
  1 | Шоколад      |   10
  2 | Принтер      | 3000
  3 | Книга        |  500
  4 | Монитор      | 7000
  5 | Гитара       | 4000
(5 rows)

test_db=# SELECT * FROM clients;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |      
  2 | Петров Петр Петрович | Canada            |      
  3 | Иоганн Себастьян Бах | Japan             |      
  4 | Ронни Джеймс Дио     | Russia            |      
  5 | Ritchie Blackmore    | Russia            |      
(5 rows)
```
---
## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys, свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения этих операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод этого запроса.
 
Подсказка: используйте директиву `UPDATE`.

## Решение:

```bash
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE наименование='Книга') WHERE фамилия='Иванов Иван Иванович';
UPDATE 1
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE наименование='Монитор') WHERE фамилия='Петров Петр Петрович';
UPDATE 1
test_db=# UPDATE clients SET "заказ" = (SELECT id FROM orders WHERE наименование='Гитара') WHERE фамилия='Иоганн Себастьян Бах';
UPDATE 1
```
```bash
test_db=# SELECT* FROM clients WHERE заказ IS NOT NULL;
 id |       фамилия        | страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)
```
---

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Решение:
```bash
test_db=# EXPLAIN SELECT* FROM clients WHERE заказ IS NOT NULL;
                        QUERY PLAN                         
-----------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72)
   Filter: ("заказ" IS NOT NULL)
(2 rows)
```
Согласно документации, команда отображает план выполнения, созданный планировщиком PostgreSQL для указанного оператора. План выполнения показывает, как будет сканироваться таблица (таблицы), на которые ссылается инструкция, — простым последовательным сканированием, сканированием индекса и т. д. В моем конкретном случае в выводе присутствует следующая информация: *cost=0.00..18.10* - предполагаемые затраты времени на вывод первой строки...всех строк; *rows=806* - количество строк, которое будет выведено; *width=72* - предполагаемый средний размер строк. Ну и *Filter: ("заказ" IS NOT NULL)* - фильтр, по которому будет проводиться сравнение записей в таблицах.

---

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

## Решение:
*Создаю бэкап*
```bash
pg_dumpall -U baldin > /home/backup/test_db.backup
```
*Останавливаю контейнер (проверяю, что нет запущенных контейнеров)*
```bash
[admin@hw-06-02-docker ~]$ docker stop postgres
postgres
[admin@hw-06-02-docker ~]$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                     PORTS     NAMES
```
*Поднимаю новый контейнер с чистой БД*
```bash
[admin@hw-06-02-docker ~]$ docker run --rm -d -e POSTGRES_USER=baldin -e POSTGRES_PASSWORD=qwerty -e POSTGRES_DB=test_db --name postgres2 postgres:12
```
*Копирую дамп в новый контейнер*
```bash
[admin@hw-06-02-docker ~]$ docker cp postgres:/home/backup/test_db.backup backup/ && docker cp backup/test_db.backup postgres2:/home/
Successfully copied 8.7kB to /home/admin/backup/
Successfully copied 8.7kB to postgres2:/home/
```
*Восстанавливаю БД из файла*
```bash
root@690fb9fc0534:/# psql -U baldin -d test_db -f /home/test_db.backup
SET
SET
SET
psql:/home/test_db.backup:14: ERROR:  role "baldin" already exists
ALTER ROLE
CREATE ROLE
ALTER ROLE
CREATE ROLE
ALTER ROLE
You are now connected to database "template1" as user "baldin".
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
You are now connected to database "postgres" as user "baldin".
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
psql:/home/test_db.backup:112: ERROR:  database "test_db" already exists
ALTER DATABASE
You are now connected to database "test_db" as user "baldin".
SET
SET
SET
SET
SET
 set_config 
------------
 
(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
CREATE TABLE
ALTER TABLE
CREATE SEQUENCE
ALTER TABLE
ALTER SEQUENCE
ALTER TABLE
ALTER TABLE
COPY 5
COPY 5
 setval 
--------
      1
(1 row)

 setval 
--------
      1
(1 row)

ALTER TABLE
ALTER TABLE
CREATE INDEX
ALTER TABLE
GRANT
GRANT
GRANT
GRANT
```
*В результате*
```bash
root@690fb9fc0534:/# psql -U baldin -d test_db
psql (12.15 (Debian 12.15-1.pgdg110+1))
Type "help" for help.

test_db=# \l
                              List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    | Access privileges 
-----------+--------+----------+------------+------------+-------------------
 postgres  | baldin | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | baldin | UTF8     | en_US.utf8 | en_US.utf8 | =c/baldin        +
           |        |          |            |            | baldin=CTc/baldin
 template1 | baldin | UTF8     | en_US.utf8 | en_US.utf8 | =c/baldin        +
           |        |          |            |            | baldin=CTc/baldin
 test_db   | baldin | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)

test_db=# 
```
