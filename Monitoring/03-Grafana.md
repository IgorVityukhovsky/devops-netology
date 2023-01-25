# Домашнее задание к занятию "10.03. Grafana"

## Задание повышенной сложности

**В части задания 1** не используйте директорию [help](./help) для сборки проекта, самостоятельно разверните grafana, где в 
роли источника данных будет выступать prometheus, а сборщиком данных node-exporter:
- grafana
- prometheus-server
- prometheus node-exporter

За дополнительными материалами, вы можете обратиться в официальную документацию grafana и prometheus.

В решении к домашнему заданию приведите также все конфигурации/скрипты/манифесты, которые вы 
использовали в процессе решения задания.

**В части задания 3** вы должны самостоятельно завести удобный для вас канал нотификации, например Telegram или Email
и отправить туда тестовые события.

В решении приведите скриншоты тестовых событий из каналов нотификаций.

## Обязательные задания

### Задание 1
Используя директорию [help](./help) внутри данного домашнего задания - запустите связку prometheus-grafana.

Зайдите в веб-интерфейс графана, используя авторизационные данные, указанные в манифесте docker-compose.

Подключите поднятый вами prometheus как источник данных.

Решение домашнего задания - скриншот веб-интерфейса grafana со списком подключенных Datasource.

## Задание 2
Изучите самостоятельно ресурсы:
- [promql-for-humans](https://timber.io/blog/promql-for-humans/#cpu-usage-by-instance)
- [understanding prometheus cpu metrics](https://www.robustperception.io/understanding-machine-cpu-usage)

Создайте Dashboard и в ней создайте следующие Panels:
- Утилизация CPU для nodeexporter (в процентах, 100-idle)
- CPULA 1/5/15
- Количество свободной оперативной памяти
- Количество места на файловой системе

Для решения данного ДЗ приведите promql запросы для выдачи этих метрик, а также скриншот получившейся Dashboard.

## Задание 3
Создайте для каждой Dashboard подходящее правило alert (можно обратиться к первой лекции в блоке "Мониторинг").

Для решения ДЗ - приведите скриншот вашей итоговой Dashboard.

## Задание 4
Сохраните ваш Dashboard.

Для этого перейдите в настройки Dashboard, выберите в боковом меню "JSON MODEL".

Далее скопируйте отображаемое json-содержимое в отдельный файл и сохраните его.

В решении задания - приведите листинг этого файла.

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

### Выполнение

### Задание 1

![Image](https://i.ibb.co/LxMBbJL/Screenshot-from-2023-01-24-16-45-06.png)


### Задание 2

- Утилизация CPU для nodeexporter (в процентах, 100-idle)  
```
100 - (avg by (instance)(irate(node_cpu_seconds_total{instance="nodeexporter:9100",mode="idle"}[5m])) * 100)
```
![Image](https://i.ibb.co/DYNCG0P/Screenshot-from-2023-01-25-12-54-03.png)

- CPULA 1/5/15
```
avg_over_time(node_load1{instance='nodeexporter:9100'}[1m])
```
```
avg_over_time(node_load1{instance='nodeexporter:9100'}[5m])
```
```
avg_over_time(node_load1{instance='nodeexporter:9100'}[15m])
```
![Image](https://i.ibb.co/Qfrp3Rh/Screenshot-from-2023-01-25-12-55-20.png)

- Количество свободной оперативной памяти
```
node_memory_MemFree_bytes
```
![Image](https://i.ibb.co/k4BZvBz/Screenshot-from-2023-01-25-12-52-22.png)

- Количество места на файловой системе
```
node_filesystem_avail_bytes {fstype=~"ext4|xfs"}
```
![Image](https://i.ibb.co/Y77HMgJ/Screenshot-from-2023-01-25-13-01-58.png)

### Задание 2

![Image](https://i.ibb.co/hBytK0P/Screenshot-from-2023-01-25-12-41-44.png)
