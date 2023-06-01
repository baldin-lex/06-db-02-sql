# Домашнее задание к занятию 2. «SQL» - Балдин

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose-манифест.

## Решение:

*Содержимое docker-compose.yaml*
```
version: '3'

volumes:
  data: {}
  backup: {}

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
```
*Запускаю:*
```console
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
```console
[admin@hw-06-02-docker ~]$ sudo docker exec -it postgres bash
root@f5b11e40a0c6:/# 
```
*Подключаюсь к базе:*
```console
baldin@42a9c452187f:/$ psql test_db -U baldin
psql (12.15 (Debian 12.15-1.pgdg110+1))
Type "help" for help.

test_db=# 
```


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
- описание таблиц (describe);
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db;
- список пользователей с правами над таблицами test_db.

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

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните, что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. задачу 1).

Остановите контейнер с PostgreSQL, но не удаляйте volumes.

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
