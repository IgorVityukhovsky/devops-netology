# Домашнее задание к занятию "7.3. Основы и принцип работы Терраформ"

## Задача 1. Создадим бэкэнд в S3 (необязательно, но крайне желательно).

Если в рамках предыдущего задания у вас уже есть аккаунт AWS, то давайте продолжим знакомство со взаимодействием
терраформа и aws. 

1. Создайте s3 бакет, iam роль и пользователя от которого будет работать терраформ. Можно создать отдельного пользователя,
а можно использовать созданного в рамках предыдущего задания, просто добавьте ему необходимы права, как описано 
[здесь](https://www.terraform.io/docs/backends/types/s3.html).
1. Зарегистрируйте бэкэнд в терраформ проекте как описано по ссылке выше. 


## Задача 2. Инициализируем проект и создаем воркспейсы. 

1. Выполните `terraform init`:
    * если был создан бэкэнд в S3, то терраформ создат файл стейтов в S3 и запись в таблице 
dynamodb.
    * иначе будет создан локальный файл со стейтами.  
1. Создайте два воркспейса `stage` и `prod`.
1. В уже созданный `aws_instance` добавьте зависимость типа инстанса от вокспейса, что бы в разных ворскспейсах 
использовались разные `instance_type`.
1. Добавим `count`. Для `stage` должен создаться один экземпляр `ec2`, а для `prod` два. 
1. Создайте рядом еще один `aws_instance`, но теперь определите их количество при помощи `for_each`, а не `count`.
1. Что бы при изменении типа инстанса не возникло ситуации, когда не будет ни одного инстанса добавьте параметр
жизненного цикла `create_before_destroy = true` в один из рессурсов `aws_instance`.
1. При желании поэкспериментируйте с другими параметрами и рессурсами.

В виде результата работы пришлите:
* Вывод команды `terraform workspace list`.
* Вывод команды `terraform plan` для воркспейса `prod`.  

---

### Ответ
ДЗ буду выполнять в яндекс облаке, ссылка на документацию по работе с Yandex Object Storage

https://cloud.yandex.ru/docs/storage/quickstart?from=int-console-help-center-or-nav


Создал s3 хранилище в веб-интерфейсе яндекс облака, назвал его s3-netology-mystate.

В параметрах указал стандартное и публичное, 1гб.

В веб-интерфейсе загрузил файл terraform.tfstate

Дал аккаунту srv-netology полный доступ к всему хранилищу

Следую документации дальше, нам необходим раздел настройки бекенда

https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-state-storage#set-up-backend

На данном этапе я имею файл provider.tf следующего содержания

```
# Provider
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}

provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id  = "${var.yandex_cloud_id}"
  folder_id = "${var.yandex_folder_id}"
}

```
Создадим iam ключи доступа, что бы в дальнейшем использовать их для доступа к s3 бакет.
```
yc iam access-key create --service-account-id #указываем наш сервис-аккаунт ID. Узнать можно с помощью yc iam access-key create
```
Из вывода этой команды для access key будет использовано значение key_id, а для secret_key будет использовано значение secret


В инструкции предлагается вставить описание бекенда в блок терраформ в следующем виде
```
backend "s3" {
    endpoint   = "storage.yandexcloud.net"
    bucket     = "s3-netology-mystate"
    region     = "ru-central1"
    key        = "terraform.tfstate"
    backend-config = "backend.conf"
    access_key = 
    secret_key = 

    skip_region_validation      = true
    skip_credentials_validation = true
  }
```
Однако в блоке terraform нельзя использовать переменные, поэтому значения ключей мы выведем в отдельный файл backend.conf и не будем выкладывать его на гитхаб. Я не нашел способа, который бы не использовал сложные конструкции из костылей и вместе с тем был бы безопасен.

В итоге наш файл provider.tf будет выглядеть следующим образом 
```
# Provider
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  backend "s3" {
    endpoint   = "storage.yandexcloud.net"
    bucket     = "s3-netology-mystate"
    #region     = "ru-central1"
    key        = "terraform.tfstate"
    backend-config = "backend.conf"
    
    skip_region_validation      = true
    skip_credentials_validation = true
  }
}

provider "yandex" {
  service_account_key_file = "key.json"
  cloud_id  = "${var.yandex_cloud_id}"
  folder_id = "${var.yandex_folder_id}"
}
```





