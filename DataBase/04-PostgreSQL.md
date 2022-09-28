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

## Ответ

Создал директорию для выполнения ДЗ со следующим содержимым:
**Вставить скриншот**
Заранее скачал бекап для следующего задания, что бы сразу подключить его к машине.
Docker-compose файл следующего содержания:
```
version: '3.8'
services:
  mydb:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
      - /home/igor/HomeWorkPostgreSQL/backup:/var/lib/postgresql/backup
    ports:
      - "5432:5432"
    environment:
      PGDATA: /var/lib/postgresql/data/
      POSTGRES_PASSWORD: root

volumes:
  db-data:
  db-backup:
networks:
  postgresnetwork000:
    driver: 'local'
```
Выполняем сборку проекта, поднимаем докер контейнер, проверяем, что он запустился
```
cd /home/igor/HomeWorkPostgreSQL
docker compose build
docker compose up -d
docker compose ps
```
Можем так же зайти в контейнер и проверить, что нужный волюм подключился и бекап находится в нём
```
docker exec -it homeworkpostgresql-mydb-1 bash
root@84b40c3158b1:/# cd /var/lib/postgresql/backup
root@84b40c3158b1:/var/lib/postgresql/backup# ls
test_dump.sql
```
Подключаемся к постгресу и выводим справку
```
psql -U postgres
\?
```
вывод списка БД
```
\db[+]  [PATTERN]      list tablespaces
```
подключение к БД
```
                         connect to new database (currently "postgres")
#Имеется ввиду, что для подключения к базе достаточно ввести её название 
```
вывод списка таблиц
```
\d[S+]                 list tables, views, and sequences
```
вывод описания содержимого таблиц
```
\d[S+]  NAME           describe table, view, sequence, or index
```
выход из psql
```
\q                     quit psql
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

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

---
