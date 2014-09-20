Пособие по Vagrant
================

Общение с узлами
------------------

Теперь мы готовы. Давайте поиграем с уже знакомой нам командой из прошлого раздела: `ansible`. Эта команда – одна из трех команд, которую Ansible использует для взаимодействия с узлами.

# Сделаем что-нибудь полезное

В прошлой команде `-m ping` означал "используй модуль _ping_". Это один из множества модулей, доступных в Ansible. Модуль `ping` очень прост, он не требует никаких аргументов. Модули, требующие аргументов, могут получить их через `-a`. Давайте взглянем на несколько модулей.

## Модуль shell

Этот модуль позволяет запускать shell-команды на удаленном узле:

    ansible -i step-02/hosts -m shell -a 'uname -a' host0.example.org

Вывод должен быть вроде:

    host0.example.org | success | rc=0 >>
    Linux host0.example.org 3.2.0-23-generic-pae #36-Ubuntu SMP Tue Apr 10 22:19:09 UTC 2012 i686 i686 i386 GNU/Linux

Легко!

## Модуль copy 

Не удивительно, модуль copy позволяет копировать файл из управляющей машины на удаленный узел. Представим что нам нужно скопировать наш `/etc/motd` в `/tmp` узла:

    ansible -i step-02/hosts -m copy -a 'src=/etc/motd dest=/tmp/' host0.example.org

Вывод:

    host0.example.org | success >> {
        "changed": true, 
        "dest": "/tmp/motd", 
        "group": "root", 
        "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
        "mode": "0644", 
        "owner": "root", 
        "size": 0, 
        "src": "/root/.ansible/tmp/ansible-1362910475.9-246937081757218/motd", 
        "state": "file"
    }

Ansible (точнее, модуль _copy_, запущенный на узле) ответил кучей полезной информации в формате JSON. Мы увидим как это можно будет использовать позже.

У Ansible есть огромный
[список модулей](http://docs.ansible.com/list_of_all_modules.html), который покрывает практически все, что можно делать в системе. Если вы не нашли подходящего модуля, написание своего модуля – довольно простая задача (и не обязательно использовать Python, нужно лишь разговаривать с помощью JSON).

# Много хостов, одна команда

Ок, все что было выше – прикольно, но нам нужно управлять множеством хостов. Давайте попробуем.

Допустим, мы хотим собрать факты про узел и, например, хотим узнать какая версия Ubuntu установлена на узлах. Это довольно легко:

    ansible -i step-02/hosts -m shell -a 'grep DISTRIB_RELEASE /etc/lsb-release' all

`all` это синоним 'все хосты в inventory-файле'. Вывод будет примерно таким:

    host1.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

    host2.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

    host0.example.org | success | rc=0 >>
    DISTRIB_RELEASE=12.04

# Больше фактов

Легко и просто. Однако, если нам нужно больше информации (IP-адреса, размеры ОЗУ, и пр.), такой подход может быстро оказаться неудобным. Решение – использовать модуль `setup`. Он специализируется на сборке _фактов_ с узлов.

Попробуйте:

    ansible -i step-02/hosts -m setup host0.example.org

ответ:

    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.0.60"
        ], 
        "ansible_all_ipv6_addresses": [], 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "01/01/2007", 
        "ansible_bios_version": "Bochs"
        },
        ---snip---
        "ansible_virtualization_role": "guest", 
        "ansible_virtualization_type": "kvm"
    }, 
    "changed": false, 
    "verbose_override": true

Вывод был сокращен для простоты, но вы можете узнать много интересного из этой информации. Вы также можете фильтровать ключи если вас интересует что-то конкретное.

Например, вам нужно узнать сколько памяти доступно на всех хостах. Это легко, запустите `ansible -i step-02/hosts -m setup -a 'filter=ansible_memtotal_mb' all`:

    host2.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

    host1.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

    host0.example.org | success >> {
        "ansible_facts": {
            "ansible_memtotal_mb": 187
        }, 
        "changed": false, 
        "verbose_override": true
    }

Заметьте, что узлы ответили не в том порядке, в котором они отвечали выше. Ansible общается с хостами параллельно!

Кстати, при использовании модуля `setup` можно указывать `*` в выражении `filter=`. Как в shell.

# Выбор хостов

Мы видели, что `all` означает 'все хосты', но в Ansible есть 
[куча иных способов выбирать хосты](http://ansible.cc/docs/patterns.html#selecting-targets):

- `host0.example.org:host1.example.org` будет запущен на host0.example.org и на
  host1.example.org
- `host*.example.org` будет запущен на всех хостах, названия которых начинается с 'host' и заканчивается на 
'.example.org' (тоже как в shell)

Есть другие способы с использованием групп, о них мы узнаем в следующем шаге 
[step-03](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-03).


