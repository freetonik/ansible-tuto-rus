Пособие по Vagrant
================

Добавляем еще один веб-сервер
-------------------------

У нас есть один веб-сервер. Мы хотим два.

# Обновление inventory

Мы ожидаем наплыва трафика, так что давайте добавим еще один веб-сервер и балансировщик, который мы настроем в следующем шаге. Давайте закончим с inventory:

    [web]
    host1.example.org ansible_ssh_host=192.168.33.11 ansible_ssh_user=root
    host2.example.org ansible_ssh_host=192.168.33.12 ansible_ssh_user=root

    [haproxy]
    host0.example.org ansible_ssh_host=192.168.33.10 ansible_ssh_user=root

Помните, здесь мы указываем `ansible_ssh_host` потому что хост имеет не тот IP что ожидается. Можно добавить эти хосты к себе в `/etc/hosts` или использовать реальные имена (что вы и будете делать в обычной ситуации).

# Сборка другого веб-сервера

Деплой второго сервера очень прост:

    $ ansible-playbook -i step-09/hosts step-09/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host2.example.org]
    ok: [host1.example.org]

    TASK: [Updates apt cache] ********************* 
    ok: [host1.example.org]
    ok: [host2.example.org]

    TASK: [Installs necessary packages] ********************* 
    ok: [host1.example.org] => (item=apache2,libapache2-mod-php5,git)
    changed: [host2.example.org] => (item=apache2,libapache2-mod-php5,git)

    TASK: [Push future default virtual host configuration] ********************* 
    ok: [host1.example.org]
    changed: [host2.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host2.example.org]
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    changed: [host2.example.org]
    changed: [host1.example.org]

    TASK: [Rolling back - Restoring old default virtualhost] ********************* 
    skipping: [host1.example.org]
    skipping: [host2.example.org]

    TASK: [Rolling back - Removing out virtualhost] ********************* 
    skipping: [host1.example.org]
    skipping: [host2.example.org]

    TASK: [Rolling back - Ending playbook] ********************* 
    skipping: [host1.example.org]
    skipping: [host2.example.org]

    TASK: [Deploy our awesome application] ********************* 
    ok: [host1.example.org]
    changed: [host2.example.org]

    TASK: [Deactivates the default virtualhost] ********************* 
    changed: [host1.example.org]
    changed: [host2.example.org]

    TASK: [Deactivates the default ssl virtualhost] ********************* 
    changed: [host2.example.org]
    changed: [host1.example.org]

    NOTIFIED: [restart apache] ********************* 
    changed: [host1.example.org]
    changed: [host2.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=10   changed=5    unreachable=0    failed=0    
    host2.example.org              : ok=10   changed=8    unreachable=0    failed=0    

Все что нужно это удалить `-l host1.example.org` из командной строки. Помните, `-l` позволяет ограничить хосты для запуска. Теперь ограничения не требуется и запуск произойдет на всех машинах группы `web`.

Если бы в группе `web` были бы другие машины, и нам нужно было бы запустить плейбук только на некоторых из них, можно было бы использовать, например, такое: `-l firsthost:secondhost:...`.

Теперь у нас есть эта милая ферма веб-серверов, давайте превратим ее в кластер с помощью балансировщика нагрузок в шаге [step-10](https://github.com/leucos/ansible-tuto/tree/master/step-10).
