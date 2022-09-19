Установка Docker copmose
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

Проверка Docker copmose
```
docker compose version
```
Создадим каталоги и дадим права
```
sudo mkdir -p /var/lib/postgresql/data
sudo mkdir -p /var/lib/postgresql/backup
sudo chmod 666 /var/lib/postgresql/data
sudo chmod 666 /var/lib/postgresql/backup
```

Создадим docker compose file
```
version: '3.8'
services:
  mydb:
    image: postgres:12
    volumes:
      - db-data:/var/lib/postgresql/data
      - db-backup:/var/lib/postgresql/backup
    ports:
      - "5432:5432"
    environment:
      PGDATA: /var/lib/postgresql/data/
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
