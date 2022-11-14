Создал рабочую директорию, куда поместил папку "Playbook" со всем содержимым
```
igor@igor-ubuntu:~/ansible/01-base/playbook$ ansible-playbook -i inventory/test.yml site.yml
```
Был выдан факт "12"
```
TASK [Print fact] **********************************************************************************
ok: [localhost] => {
    "msg": 12
}
```
Указанное значение находилось в 01-base/playbook/group_vars/all/examp.yml. Поменял на all default fact. Повторил команду, получил вывод
```
ASK [Print fact] **********************************************************************************
ok: [localhost] => {
    "msg": "all default fact"
}
```
