Устанавливаем утилиту яндекс-клауда **yc**

```
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
```
restart bash

install yc

yc init

#for get token https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb

yc compute image list

mkdir packer
cd packer

yc vpc network create --name net && yc vpc subnet create --name my-subnet-a --zone ru-central1-a --range 10.1.2.0/24 --network-name net --description "my first subnet via yc"

sudo nano centos-7-base.json

{
  "builders": [
    {
      "disk_type": "network-nvme",
      "folder_id": "b1gaec42k169jqpo02f7",         #Change for my
      "image_description": "by packer",
      "image_family": "centos",
      "image_name": "centos-7-base",
      "source_image_family": "centos-7",
      "ssh_username": "centos",
      "subnet_id": "e9b28580oai8e7qo4p13",         #Change for my 
      "token": "",                                 #Change for my
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

install packer:

Install VPN plugin for browser
Dawnloaded binary file with official sute
Copy in $PATH
sudo mv ~/Downloads/packer /usr/local/bin/
restart bash

In packer folders create file config.pkr.hcl

packer {
  required_plugins {
    yandex = {
      version = ">= 1.1.2"
      source  = "github.com/hashicorp/yandex"
    }
  }
}

cd packer
packer init config.pkr.hcl

packer build centos-7-base.json


yc compute image list
See that image created

in terraform/variables.tf change ID on my

Unstall terraform

sudo apt install snapd
sudo apt update
sudo snap install terraform --classic

create service account (for example in web-interface yandex cloude)
check:
yc iam service-account --folder-id <ID каталога> list

Создайте авторизованный ключ для сервисного аккаунта и сохраните его в файл key.json
cd terraform
yc iam key create --service-account-name default-sa --output key.json --folder-id <ID каталога>

yc config profile create sa-profile
yc config set service-account-key key.json

check:
yc config list

yc config set cloud-id <ID облака>
yc config set folder-id <ID каталога>

before creating VM without terraform need delete temp lan for packer:
yc vpc subnet delete --name my-subnet-a && yc vpc network delete --name net

settings terraform:
cd terraform
nano ~/.terraformrc

provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}

terraform init

generate ssh-keys for nodes:
ssh-keygen


terraform plan
terraform apply -auto-approve
(paste screeshot)

paste external IP in file /ansible/inventory

cd
cd ansible
ansible-playbook provision.yml

For task 4

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
