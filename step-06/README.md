Пособие по Vagrant
================

Перезапуск в случае ошибки конфигурации
------------------------------

Мы установили apache, изменили virtualhot и перезапустили сервер. Но что если мы хотим перезапускать сервер только если конфигурация корректна? 

# Откатываемся если есть проблемы

Ansible содержит ловкую штуку: он остановит всю обработку если что-то пошло не так. Мы используем эту особенность чтобы остановить плейбук когда конфигурация не валидна.

Давайте изменим файл конфигурации виртуального хоста `awesome-app` и сломаем его:

    <VirtualHost *:80>
      RocumentDoot /var/www/awesome-app

      Options -Indexes

      ErrorLog /var/log/apache2/error.log
      TransferLog /var/log/apache2/access.log
    </VirtualHost>

Как я сказал, если задача не может исполнится, обработка останавливается. Так что нужно удостовериться в валидности конфигурации перед перезапуском сервереа. Мы также начнем с добавления виртуального хоста _до_ удаления дефолтного виртуального хоста, так что последующий перезапуск (возможно, сделанный напрямую на сервере) не сломает apache.

Нужно было сделать это в самом начале. Так как мы уже запускали этот плейбук, дефолтный виртуальный хост уже деактивирован. Не проблема: этот плейбук можно использовать на других невинных хостах, так что давайте защитим их.

    - hosts: web
      tasks:
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Push future default virtual host configuration
          copy: src=files/awesome-app dest=/etc/apache2/sites-available/ mode=0640

        - name: Activates our virtualhost
          command: a2ensite awesome-app

        - name: Check that our config is valid
          command: apache2ctl configtest

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

    $ ansible-playbook -i step-06/hosts -l host1.example.org step-06/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push future default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    failed: [host1.example.org] => {"changed": true, "cmd": ["apache2ctl", "configtest"], "delta": "0:00:00.045046", "end": "2013-03-08 16:09:32.002063", "rc": 1, "start": "2013-03-08 16:09:31.957017"}
    stderr: Syntax error on line 2 of /etc/apache2/sites-enabled/awesome-app:
    Invalid command 'RocumentDoot', perhaps misspelled or defined by a module not included in the server configuration
    stdout: Action 'configtest' failed.
    The Apache error log may have more information.

    FATAL: all hosts have already failed -- aborting

    PLAY RECAP ********************* 
    host1.example.org              : ok=4    changed=2    unreachable=0    failed=1    

Как вы заметили, `apache2ctl` возвращает код ошибки 1. Ansible видит это и останавливает работу. Отлично!

Ммм, хотя нет, не отлично... Наш виртуальный хост все равно был добавлен. При любой последующей попытке перезапуска Apache будет ругаться на конфигурацию и выключаться. Так что нам нужен способ отлавливать ошибки и возвращаться к рабочему состоянию. 

Давайте займемся этим в [step-07](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-07).
