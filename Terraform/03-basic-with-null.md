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
sudo nano /etc/resolv.conf
```
Приведём строчку к виду
```
nameserver 8.8.8.8
```
Скачаем актуальную версию терраформ
```
curl -o https://hashicorp-releases.yandexcloud.net/terraform/1.3.3/terraform_1.3.3_linux_amd64.zip
```
