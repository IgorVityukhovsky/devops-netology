Установите Bitwarden плагин для браузера. Зарегистрируйтесь и сохраните несколько паролей.
```
выполнено
```

Установите Google authenticator на мобильный телефон. Настройте вход в Bitwarden аккаунт через Google authenticator OTP
```
выполнено
```
Установите apache2, сгенерируйте самоподписанный сертификат, настройте тестовый сайт для работы по HTTPS.
```
sudo apt install apache2
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
```
Generating a RSA private key...
...writing new private key to '/etc/ssl/private/apache-selfsigned.key'
```
