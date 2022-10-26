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
Добавим аутентификационные данные в переменные окружения и прочтём их
```
echo "export YC_TOKEN=$(yc iam create-token)" >> /etc/environment && source /etc/environment
echo "export YC_CLOUD_ID=$(yc config get cloud-id)" >> /etc/environment && source /etc/environment
echo "export YC_FOLDER_ID=$(yc config get folder-id)" >> /etc/environment && source /etc/environment
```




