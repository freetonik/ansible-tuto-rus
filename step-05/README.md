Пособие по Vagrant
================

Улучшаем набор apache
---------------------

Мы установили apache, давайте теперь настроим virtualhost.

# Улучшение плейбука

Нам нужен лишь один виртуальный хост на сервере, но мы хотим сменить дефолтный на что-то более конкретное. Поэтому нам придется удалить текущий virtualhost, отправить наш virtualhost, активировать его и перезапустить apache. 

Давайте создадим директиорию под названием `files` и добавим нашу конфигурацию для host1.example.org, назовем ее `awesome-app`:

    <VirtualHost *:80>
      DocumentRoot /var/www/awesome-app

      Options -Indexes

      ErrorLog /var/log/apache2/error.log
      TransferLog /var/log/apache2/access.log
    </VirtualHost>

Теперь небольшой обноление плейбука и все готово:

    - hosts: web
      tasks:
        - name: Installs apache web server
          apt: pkg=apache2 state=installed update_cache=true

        - name: Push default virtual host configuration
          copy: src=files/awesome-app dest=/etc/apache2/sites-available/ mode=0640 

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl

        - name: Activates our virtualhost
          command: a2ensite awesome-app
          notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

Поехали:

    $ ansible-playbook -i step-05/hosts -l host1.example.org step-05/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push default virtual host configuration] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Deactivates the default ssl virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    NOTIFIED: [restart apache] ********************* 
    changed: [host1.example.org]

    PLAY RECAP ********************* 
    host1.example.org              : ok=7    changed=5    unreachable=0    failed=0    

Круто! Ну, если задуматсья, мы немного опережаем события. Не нужно ли проверить корректность конфигурации перед тем, как перезапускать apache? Чтобы не нарушать работоспособность сервиса в случае если конфигурация содержит ошибку.

Давайте сделаем это в [step-06](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-06).
