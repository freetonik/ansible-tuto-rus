Пособие по Vagrant
================

Деплоим сайт с помощью Git
------------------------------

Мы установили Apache, добавили виртуальный хост и безопасно перезапустили сервер. Теперь давайте используем модуль git чтобы сделать деплой приложения.

# Модуль git

Ну, честно говоря, тут все будет просто, ничего нового. Модуль `git` это просто еще один модуль. Но давайте попробуем что-нибудь интересное. А позже это пригодится когда мы будем работать с `ansible-pull`.

Виртуальный хост задан, но нам нужно внести пару изменений чтобы закончить деплой. Мы деплоим приложение на PHP, так что нужно установить пакет `libapache2-mod-php5`. Также нужно установить сам `git`, так как, очевидно, модуль git требует его наличия.

Можно сделать так:

        ...
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Installs php5 module
          apt: pkg=libapache2-mod-php5 state=installed

        - name: Installs git
          apt: pkg=git state=installed
        ...

но в Ansible есть способ лучше. Он может проходить по набору элементов и использовать каждый в определенном действии, вот так:

    - hosts: web
      tasks:
        - name: Updates apt cache
          apt: update_cache=true

        - name: Installs necessary packages
          apt: pkg={{ item }} state=latest 
          with_items:
            - apache2
            - libapache2-mod-php5
            - git

        - name: Push future default virtual host configuration
          copy: src=files/awesome-app dest=/etc/apache2/sites-available/ mode=0640

        - name: Activates our virtualhost
          command: a2ensite awesome-app

        - name: Check that our config is valid
          command: apache2ctl configtest
          register: result
          ignore_errors: True

        - name: Rolling back - Restoring old default virtualhost
          command: a2ensite default
          when: result|failed

        - name: Rolling back - Removing out virtualhost
          command: a2dissite awesome-app
          when: result|failed

        - name: Rolling back - Ending playbook
          fail: msg="Configuration file is not valid. Please check that before re-running the playbook."
          when: result|failed

        - name: Deploy our awesome application
          git: repo=https://github.com/leucos/ansible-tuto-demosite.git dest=/var/www/awesome-app
          tags: deploy

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl
          notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted


Поехали:

    $ ansible-playbook -i step-08/hosts -l host1.example.org step-08/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Updates apt cache] ********************* 
    ok: [host1.example.org]

    TASK: [Installs necessary packages] ********************* 
    changed: [host1.example.org] => (item=apache2,libapache2-mod-php5,git)

    TASK: [Push future default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Restoring old default virtualhost] ********************* 
    skipping: [host1.example.org]

    TASK: [Rolling back - Removing out virtualhost] ********************* 
    skipping: [host1.example.org]

    TASK: [Rolling back - Ending playbook] ********************* 
    skipping: [host1.example.org]

    TASK: [Deploy our awesome application] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default ssl virtualhost] ********************* 
    changed: [host1.example.org]

    NOTIFIED: [restart apache] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=10   changed=8    unreachable=0    failed=0    

Теперь можно перейти на [http://192.168.33.11](http://192.168.33.11) и увидеть котенка и имя сервера.

Строка `tags: deploy позволяет запустить определнную порцию плейбука. Допустим, вы запушили новую версию сайта. Вы хотите ускорить и запустить только ту часть, которая ответственна за деплой. Это можно сделать с помощью тегов. Конечно, "deploy" это просто строка, можно задавать любую. Давайте посмотрим как это можно использовать:

    $ ansible-playbook -i step-08/hosts -l host1.example.org step-08/apache.yml -t deploy 
    X11 forwarding request failed on channel 0

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Deploy our awesome application] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=2    changed=1    unreachable=0    failed=0    

Ок, давайте задеплоим другой веб-сервер в шаге [step-09](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-09).
