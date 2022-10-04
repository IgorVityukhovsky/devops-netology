# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib` 
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled` 
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

## Ответ

Подготовим машину для работы, увеличив количество выделяемой виртуальной памяти
```
sudo sysctl -w vm.max_map_count = 262144
```
Для решения задачи использовал docker compose
```
version: '3.8'
services:
  elasticsearch:
    image: elasticsearch:7.17.6   #Не было тага 7, использовал 7.17.6
    container_name: es
    tty: true
    stdin_open: true
    ulimits:
     nofile:
      soft: 262144
      hard: 262144

    entrypoint: /bin/bash -c "chmod 777 /var/lib && mkdir /usr/share/elasticsearch/snapshots && chmod 777 /usr/share/elasticsearch/snapshots && /bin/tini "/usr/local/bin/docker-entrypoint.sh eswrapper""

    #Команда, которая выполнится при запуске контейнера, задаёт необходимые права на нужную нам директорию
    #Если оставить только команду на назначение прав, контейнер будет считать её основной и после её завершения завершится и контейнер
    #Что бы этого избежать, добавил в команду запуск процесса elasticsearch так как именно он нам и нужен как основной
    #Команда взята из столбца COMMAND при запуске дефолтного контейнера
    #Так же накидывает нужные права на папку из задачи 3
    

    environment:
      #- ES_HEAP_SIZE=2200m
      #- LS_HEAP_SIZE=1100m
      - discovery.type=single-node                     #Необходимо для работы одной ноды
      - node.name=netology_test                        #Задаём имя ноды
      - path.data=/var/lib                             #Задаём путь для хранения данных
      - path.repo=/usr/share/elasticsearch/snapshots   #Задаём путь для репозитория снапшотов для задачи 3
      #- xpack.security.enabled=true
      - "ES_JAVA_OPTS=-Xms3g -Xmx3g"
      - "ES_HEAP_SIZE=4g"

networks:
  elasticsearch:
    driver: 'local'
```    

Собираем проект, запускаем контейнер, проверяем, что он запустился, подключаемся к нему и выполняем нужный нам запрос /
```
docker compose build
docker compose up -d
docker compose ps
docker exec -it es bash
curl -XGET 'http://localhost:9200/'
```
```
{
  "name" : "netology_test",                  #Имя которое мы задали
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "00kuogNgSki7om2uwB8Q3w",
  "version" : {
    "number" : "7.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "f65e9d338dc1d07b642e14a27f338990148ee5b6",
    "build_date" : "2022-08-23T11:08:48.893373482Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
Так же можем проверить, что остальные наши настройки так же применились
```
curl -XGET 'http://localhost:9200/_nodes'
```
![скриншот](https://i.ibb.co/zNXz9Vf/Screenshot-from-2022-10-04-14-21-08.png) 
![скриншот](https://i.ibb.co/xfs8tRZ/Screenshot-from-2022-10-04-19-56-29.png)


В будущем объяснить про /elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml


## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
