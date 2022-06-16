Задача: Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.

Сделал: git log aefea -n 1

Результат: 

commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545
Update CHANGELOG.md




Задача: Какому тегу соответствует коммит 85024d3?

Сделал: git log 85024d3 -n 1

Результат:

(tag: v0.12.23)




Задача: Сколько родителей у коммита b8d720? Напишите их хеши.

Сделал: git log b8d720 -n 1

Результат:

#два родителя, их сокращенные хеши:
Merge: 56cd7859e 9ea88f22f




Задача: Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.

Сделал: git rev-list --ancestry-path v0.12.23..v0.12.24

Результат: 

33ff1c03bb960b332be3af2e333462dde88b279e
b14b74c4939dcab573326f4e3ee2a62e23e12f89
3f235065b9347a758efadc92295b540ee0a5e26e
6ae64e247b332925b872447e9ce869657281c2bf
5c619ca1baf2e21a155fcdb4c264cc9e24a2a353
06275647e2b53d97d4f0a19a0fec11f6d69820b5
d5f9411f5108260320064349b757f55c09bc4b80
4b6d06cc5dcb78af637bbb19c198faff37a066ed
dd01a35078f040ca984cdd349f18d0b67e486c35
225466bc3e5f35baa5d07197bbc079345b77525e




Задача: Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...)
(вместо троеточего перечислены аргументы).

Сделал: git log -S 'func providerSource'

Результат: 

commit 8c928e83589d90a031f811fae52a81be7153e82f
#самый ранний коммит с упоминанием этой функции


Задача: Найдите все коммиты в которых была изменена функция globalPluginDirs.

Сделал: git log -S 'globalPluginDirs' --oneline

Результат: 

125eb51dc Remove accidentally-committed binary
22c121df8 Bump compatibility version to 1.3.0 for terraform core release (#30988)
35a058fb3 main: configure credentials from the CLI config file
c0b176109 prevent log output during init
8364383c3 Push plugin discovery down into command package


Задача: Кто автор функции synchronizedWriters?

Сделал: git log -S 'synchronizedWriters'

Результат: 

впервые эта функция упоминается в коммите за авторством Martin Atkins


