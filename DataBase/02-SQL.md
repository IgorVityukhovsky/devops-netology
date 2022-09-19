Установка Docker copmose
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

Проверка Docker copmose
```
docker compose version
```
Создадим каталоги и дадим права, если мы хотим, что бы Volume хранились в определённых местах
```
sudo mkdir -p /var/lib/postgresql/data
sudo mkdir -p /var/lib/postgresql/backup
sudo chmod 777 /var/lib/postgresql/data
sudo chmod 777 /var/lib/postgresql/backup
```
Для нашей тестовой задачи мы будем использовать дефолтные пути (/var/lib/docker/volumes/), поэтому предыдущий пункт не обязателен.

Создадим docker compose file
```
version: '3.8'
services:
  mydb:
    image: postgres:12
    volumes:
      - db-data:/var/lib/postgresql/data
      - db-backup:/var/lib/postgresql/backup
      - /home/igor/HomeWorkSQL/Script:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    environment:
      PGDATA: /var/lib/postgresql/data/
      #POSTGRES_USER: "root"
      POSTGRES_PASSWORD: example

    #restart:
    #  no
volumes:
  db-data:
  db-backup:
networks:
  postgresnetwork000:
    driver: 'local'
```
Соберём проект
```
docker compose build 
```
Запустим проект
```
docker compose up -d
```
Проверим, что контейнер создался, запущен и работает
```
docker compose ps
```
В docker compose файле я так же подключил - /home/igor/HomeWorkSQL:/Script
В папке Script лежит скрипт StartScript, который создаёт необходимые для задач базы данных, выборки и выводит их результат в файлы.
Сделано с целью автоматизировать процесс.
Описание скрипта:
```
CREATE DATABASE test_db
	TEMPLATE template0
	ENCODING 'UTF-8';
CREATE USER "test-admin-user";
CREATE USER "test-simple-user";
CREATE TABLE orders (
	id	serial PRIMARY KEY,
	order_name	varchar(25) NOT NULL CHECK (order_name <> ''),
	price	integer NOT NULL CHECK (price > 0)
);
CREATE TABLE clients (
        id      serial PRIMARY KEY,
        last_name	varchar(45) NOT NULL CHECK (last_name <> ''),
	country	varchar(35) NOT NULL CHECK (country <> ''),
	order_number	integer REFERENCES orders
);
CREATE INDEX country_index ON clients(country);
GRANT ALL ON ALL TABLES IN SCHEMA "public" TO "test-admin-user";
GRANT SELECT, INSERT, UPDATE, DELETE
	ON ALL TABLES IN SCHEMA "public"
	TO "test-simple-user";

\o /tmp/output-task2
\l
\d+ orders
\d+ clients
SELECT table_name, grantee, privilege_type
	FROM information_schema.role_table_grants
	WHERE table_name='orders';
SELECT table_name, grantee, privilege_type
	FROM information_schema.role_table_grants
	WHERE table_name='clients';

\o
INSERT INTO orders
	VALUES (1, 'Шоколад', 10),
		(2, 'Принтер', 3000),
		(3, 'Книга', 500),
		(4, 'Монитор', 7000),
		(5, 'Гитара', 4000);
INSERT INTO clients
	VALUES (1, 'Иванов Иван Иванович', 'USA'),
		(2, 'Петров Петр Петрович', 'Canada'),
		(3, 'Иоганн Себастьян Бах', 'Japan'),
		(4, 'Ронни Джеймс Дио', 'Russia'),
		(5, 'Ritchie Blackmore', 'Russia');

\o /tmp/output-task3
SELECT * FROM orders;
SELECT * FROM clients;
SELECT COUNT(*) FROM orders;
SELECT COUNT(*) FROM clients;

\o
UPDATE clients SET order_number=3 WHERE id=1;
UPDATE clients SET order_number=4 WHERE id=2;
UPDATE clients SET order_number=5 WHERE id=3;

\o /tmp/output-task4
SELECT * FROM clients;
SELECT * FROM clients WHERE order_number IS NOT NULL;

\o /tmp/output-task5
EXPLAIN (FORMAT YAML) SELECT * FROM clients WHERE order_number IS NOT NULL;

```



Подключимся в контейнер и передаём скрипт
```
docker exec -it homeworksql-mydb-1 bash
psql -f /docker-entrypoint-initdb.d/StartScript -U postgres
```
