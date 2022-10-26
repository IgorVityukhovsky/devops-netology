Так как с предыдущих ДЗ что-то пошло не так и ничего не получалось, я решил переехать на новую машину и начать процесс с нуля, попутно его подробно документируя.

Команда для смены раскладки
```
gsettings set org.gnome.desktop.wm.keybindings switch-input-source "['<Alt>Shift_L']"
```
Установим curl
```
sudo apt install curl
```
Изменим конфиг, что бы DNS корректно резовлвились
```
sudo sh -c "echo 'nameserver 8.8.8.8' >> /etc/resolv.conf"
```
Скачаем актуальную версию терраформ
```
wget "https://hashicorp-releases.yandexcloud.net/terraform/1.3.3/terraform_1.3.3_linux_amd64.zip"
```
Разархивируем
```
unzip terraform_1.3.3_linux_amd64.zip 
```
Создадим симлинк и сделаем файлы исполняемыми
```
sudo ln -s /home/igor/terraform /usr/bin/terraform
sudo chmod ugo+x /usr/bin/terraform*
```
Установка Яндекс Клауд CLI
```
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```
Далее необходимо перезапустить терминал и проверить, что всё установилось
```
yc --version
```
Делаем симлинк
```
sudo ln -s /home/igor/yandex-cloud/bin/yc /usr/bin/yc
```

При инициализации yc CLI выйдет ссылка по которой нужно перейти, что бы узнать свой токен.

Если по каким то причинам этого не произошло можно:

Из инструкции

https://cloud.yandex.ru/docs/cli/quickstart#install
перейти по ссылке 

https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb
Получаем наш токен


Инициализируем yc CLI
```
yc init
```
Вставляем наш токен.
При дальнейших вопросах ответил, что folder по умолчанию это netology, зона ru-central1-a (криншот)

Проверить наши настройки 
```
yc config list
```
Далее необходимо создать сервисный аккаунт и присвоить ему необходимые роли. Это уже сделано и легко делается через веб-интерфейс.


Далее нас просят создать авторизованные ключи для сервисного аккаунта по инструкции из веб интерфейса или CLI. Даже если это уже было сделано ранее, лучше создать заново, так как срок его действия быстро истекает

Создал ключи в веб интерфейсе и скачал файл с ними

Создадим рабочую директорию и перейдём туда
```
mkdir myterraform && cd myterraform
```
Скопировал файл с ключаси в директорию.

Создадим профиль service-profile
```
yc config profile create service-profile
```
```
Profile 'service-profile' created and activated
```
Зададим конфигурацию профиля используя наш файл с ключами. Не забываем делать это из рабочей дирректории, где лежат ключи
```
yc config set service-account-key authorized_key.json
yc config set cloud-id b1g8rvrldf45r9h4mnbl
yc config set folder-id b1gcj17iv37qg7h91dfe  
```
Добавим аутентификационные данные в переменные окружения. Некоторые символы экранируем, что бы вставлялись команды, а не их значения
```
sudo sh -c "echo export YC_TOKEN=\$\(yc iam create-token\) >> /etc/environment"
sudo sh -c "echo export YC_CLOUD_ID=\$\(yc config get cloud-id\) >> /etc/environment"
sudo sh -c "echo export YC_FOLDER_ID=\$\(yc config get folder-id\) >> /etc/environment"
```
И прочтём их
```
source /etc/environment
```
Так же скопировал ssh ключи для машин в директорию .ssh

Настройка терраформ.

Добавим зеркала в терраформ
```
nano ~/.terraformrc
```
```
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

Создадим provider.tf
```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  zone = "ru-central1-a"
}
```
Создадим подсеть в веб-интерфейсе, а так же заполним main.tf для проверки на создание ресурсов
```
resource "yandex_compute_instance" "node01" {
  name                      = "node01"
  zone                      = "ru-central1-a"
  hostname                  = "node01.netology.cloud"
  allow_stopping_for_update = true

  resources {
    cores  = 8
    memory = 8
  }

  boot_disk {
    initialize_params {
      image_id = "fd8kb72eo1r5fs97a1ki" #ubuntu
      name     = "root-node01"
      type     = "network-nvme"
      size     = "50"
    }
  }
  network_interface {
    subnet_id = "enpenfjkmb5ge5k7817o"
    nat       = true
  }

  metadata = {
    ssh-keys = "centos:${file("~/.ssh/id_rsa.pub")}"
  }
}


```
Инициализируем терраформ
```
terraform init
```
В наш provider.tf добавим информацию о бекенде.
В итоге он будет иметь вид
```
 terraform {
  backend "s3" {
      endpoint = "storage.yandexcloud.net"
      bucket   = "s3-netology-mystate2"
      region   = "ru-central1-a"
      key      = "testfolder/terraform.tfstate"
      access_key = "YCAJEuaUsqTJ0zxKwy6o9u4ct"
      secret_key = "YCMIundsYTft-w1Fn-IxxJOpo95c6IRvuFvnIyd3"

      skip_region_validation      = true
      skip_credentials_validation = true
    }
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
    
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  zone = "ru-central1-a"
}

 
```



Создадим iam ключи доступа, что бы управлять s3 бакетом в будущем.
Можно использовать уже существующие ключи, если мы знаем их access key и secret key.
Но если мы их не знаем, секретная часть выдаётся только один раз при генерации, поэтому сгенерируем ключи
```
yc iam access-key create --service-account-id aje164j36fslfgb32qce --description "bucket key"
```
Из вывода нас интересует key_id (это наш access key) и secret (это наш secret key)
Создадим файл backend.conf и запишем туда эти данные
```
access_key = "наш ключ"
secret_key = "наш ключ подлиннее"
```
В блоке terraform нельзя использовать переменные, таким образом мы вывели секреты отдельно, что бы не хранить их на гите.
Переконфигурируем наш терраформ.
```
terraform init -reconfigure -backend-config=backend.conf
```



