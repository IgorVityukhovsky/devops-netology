Устанавливаем утилиту яндекс-клауда **yc**

```
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```
Переоткрываем bash терминал
```
install yc
```
Инициализируем
```
yc init
```
Во время инициализации нужно будет перейти по ссылке в инструкции из терминала, что бы получить токен, либо найти самостоятельно

Убеждаемся, что у нас пока нет образов в облаке яндекса
```
yc compute image list
```
Создаём директорию для Packer
```
mkdir packer
cd packer
```
Создаём сеть в яндекс облаке
```
yc vpc network create --name net && yc vpc subnet create --name my-subnet-a --zone ru-central1-a --range 10.1.2.0/24 --network-name net --description "my first subnet via yc"
```
Создаём файл на основе которого будет происходить сборка образа
```
sudo nano centos-7-base.json
```
```
{
  "builders": [
    {
      "disk_type": "network-nvme",
      "folder_id": "b1gaec42k169jqpo02f7",         Изменить значение на своё
      "image_description": "by packer",
      "image_family": "centos",
      "image_name": "centos-7-base",
      "source_image_family": "centos-7",
      "ssh_username": "centos",
      "subnet_id": "e9b28580oai8e7qo4p13",         Изменить значение на своё 
      "token": "",                                 Изменить значение на своё
      "type": "yandex",
      "use_ipv4_nat": true,
      "zone": "ru-central1-a"
    }
  ],
  "provisioners": [
    {
      "inline": [
        "sudo yum -y update",
        "sudo yum -y install bridge-utils bind-utils iptables curl net-tools tcpdump rsync telnet openssh-server"
      ],
      "type": "shell"
    }
  ]
}
```
Установим Packer.

В нынешней ситуации установил VPN плагин для браузера, скачал файл с официального сайта.
Скопировать в $PATH
```
sudo mv ~/Downloads/packer /usr/local/bin/
```
Перезапускаем bash терминал 

В дирректории Packer создаём файл config.pkr.hcl с содержимым
```
packer {
  required_plugins {
    yandex = {
      version = ">= 1.1.2"
      source  = "github.com/hashicorp/yandex"
    }
  }
}
```
Инициализируем, собираем образз, проверяем, что он создался и лежит в нашем яндекс облаке
```
cd packer
packer init config.pkr.hcl
packer build centos-7-base.json
yc compute image list
```

#Сюда вставить скриншот с образом пекера



В terraform/variables.tf меняем ID на свой

Установка Terraform.

Установил пакетный менеджер snapd и установил с помощью него, так как он подкачивает и настраивает все необходимые зависимости
```
sudo apt install snapd
sudo apt update
sudo snapd install terraform --classic
```
Создаём сервисный аккаунт (например в веб-интерфейсе яндекса)
Проверяем
```
yc iam service-account --folder-id <ID каталога> list
```

Создаём авторизованный ключ для сервисного аккаунта и сохраняем его в файл key.json
```
cd terraform
yc iam key create --service-account-name default-sa --output key.json --folder-id <ID каталога>
yc config profile create sa-profile
yc config set service-account-key key.json
```
Проверим:
```
yc config list
yc config set cloud-id <ID облака>
yc config set folder-id <ID каталога>
```
Перед созданием виртуальных машин с помощью Terraform, удаляем временную сеть, которую мы создавали для Packer
```
yc vpc subnet delete --name my-subnet-a && yc vpc network delete --name net
```
Настраиваем terraform:
```
cd terraform
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
```
terraform init
```
Генерируем ssh ключи для наших нод:
```
ssh-keygen
```
Проверяем, что собирается создать Terraform и запускаем
```
terraform plan
terraform apply -auto-approve
```
#Вставить скриншот со свойствами виртуальной ноды 1

Подставляем внешние IP в /ansible/inventory и запускаем
```
cd
cd ansible
ansible-playbook provision.yml
```
**Задача 4**

created node02.tf
changed output.tf (add node02)

terraform apply -auto-approve


Add node02 in inventory file
Add host node02 in provision.yml with different docker-compouse file

ansible-playbook provision.yml

login in VM node01
ssh centos@IP

sudo yum install nano

sudo docker cp 2f1fa28d950b:/etc/prometheus/prometheus.yml /home/centos

sudo nano prometheus.yml

add nodeexporter2 IP:port
sudo docker cp /home/centos/prometheus.yml 2f1fa28d950b:/etc/prometheus/

#connected in docker container with prometheus
#sudo docker exec -it ID /bin/sh

curl -X POST http://admin:admin@51.250.87.218:9090/-/reload

