# Домашнее задание к занятию "4.1. Командная оболочка Bash: Практические навыки"

## Обязательная задача 1

Есть скрипт:
```bash
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```

Какие значения переменным c,d,e будут присвоены? Почему?

| Переменная  | Значение | Обоснование |
| ------------- | ------------- | ------------- |
| `c`  | a+b  | Переменным может быть присвоено либо число, либо строка. "a+b" это не число, значит присвоилась строка |
| `d`  | 1+2  | Переменная d не целочисленная и так как перед переменными постален символ $ присвоились их значения в строку |
| `e`  | 3  | Благодаря $(( )) происходит присвоение целочисленной суммы "a" и "b" в "e" |


## Обязательная задача 2
На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным (после чего скрипт должен завершиться). В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
```bash
while ((1==1)
do
	curl https://localhost:4757
	if (($? != 0))
	then
		date >> curl.log
	fi
done
```

### Ваш скрипт:
```bash
while ((1==1)
do
	curl https://localhost:4757
	if (($? != 0))
	then
            LogSize=$(stat -c %s 'curl.log')
            if (($LogSize < 1048576))
            then
		date >> curl.log
            else
            date > curl.log
	fi
done
#Добавил проверку размера лог файла. При достижении в 1мб будет очищаться и перезаписываться. Таким образом лог файл не будет тяжелее 1мб.

```

## Обязательная задача 3
Необходимо написать скрипт, который проверяет доступность трёх IP: `192.168.0.1`, `173.194.222.113`, `87.250.250.242` по `80` порту и записывает результат в файл `log`. Проверять доступность необходимо пять раз для каждого узла.

### Ваш скрипт:
```bash
#!/usr/bin/env bash
declare -i a
a=0
while (($a<5))
do 
curl 192.168.0.1:80
echo $? >> curl.log
curl 173.194.222.113:80
echo $? >> curl.log
curl 87.250.250.242:80
echo $? >> curl.log
let "a +=1"
done
```

## Обязательная задача 4
Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - IP этого узла пишется в файл error, скрипт прерывается.

### Ваш скрипт:
```bash
#!/usr/bin/env bash
ip_array=(192.168.0.1:80 173.194.222.113:80 87.250.250.242:80)
while ((1==1))
  do
  for i in ${ip_array[@]}
    do
    curl $i
    if (($? != 0))
      then
      echo $i > error
    fi
  done
done

```

## Дополнительное задание (со звездочкой*) - необязательно к выполнению

Мы хотим, чтобы у нас были красивые сообщения для коммитов в репозиторий. Для этого нужно написать локальный хук для git, который будет проверять, что сообщение в коммите содержит код текущего задания в квадратных скобках и количество символов в сообщении не превышает 30. Пример сообщения: \[04-script-01-bash\] сломал хук.

### Ваш скрипт:
```bash
???
```
