curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash

restart bash

yc init

#for get token https://oauth.yandex.ru/authorize?response_type=token&client_id=1a6990aa636648e9b2ef855fa7bec2fb

yc compute image list

mkdir packer
cd packer
sudo nano centos-7-base.json

{
  "builders": [
    {
      "disk_type": "network-nvme",
      "folder_id": "b1gaec42k169jqpo02f7", #Change for my
      "image_description": "by packer",
      "image_family": "centos",
      "image_name": "centos-7-base",
      "source_image_family": "centos-7",
      "ssh_username": "centos",
      "subnet_id": "e9b28580oai8e7qo4p13",
      "token": "",
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



