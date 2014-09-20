Пособие по Vagrant
================

# Inventory

Чтобы продолжить, нам нужно подготовить inventory-файл. Место по умолчанию это `/etc/ansible/hosts`. 
Но вы можете настроить Ansible использовать другой путь, для этого используется переменная окружения (`ANSIBLE_HOSTS`) или флаг `-i`.

Мы создали такой inventory-файл:

    host0.example.org ansible_ssh_host=192.168.33.10 ansible_ssh_user=root
    host1.example.org ansible_ssh_host=192.168.33.11 ansible_ssh_user=root
    host2.example.org ansible_ssh_host=192.168.33.12 ansible_ssh_user=root

`ansible_ssh_host` это специальная _переменная_, которая содержит IP-адрес узла, к которому будет происходить соединение. В данном случае она не обязательна если вы используете gem vagrant-hostmaster. Также, вам нужно будет менять IP-адреса если вы устанавливали и настраивали свою виртуальную машину с другими адресами.

`ansible_ssh_user` это еще одна специальная _переменная_ которая говорит Ansible'у подключаться под указанным аккаунтом (юзером). По умолчанию Ansible использует ваш текущий аккаунт, или другое значение по умолчанию, указанное в ~/.ansible.cfg (`remote_user`).

# Проверка

Теперь когда Ansible установлен, давайте проверим что все работает:

    ansible -m ping all -i step-01/hosts

Здесь Ansible попытается запустить модуль `ping` (подробнее о модулях позже) на каждом хосте

Вывод должен быть примерно таким:

    host0.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

    host1.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

    host2.example.org | success >> {
        "changed": false, 
        "ping": "pong"
    }

Отлично! Все три хоста живы и здоровы, и Ansible может общаться с ними.

Теперь переходите к следующему шагу [step-02](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-02).

