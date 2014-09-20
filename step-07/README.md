Пособие по Vagrant
================

Использование условий
------------------

Мы установили Apache, добавили виртуальный хост и перезапустили сервер. Но мы хотим вернуться к рабочему состоянию если что-то пошло не так.

# Возврат при проблемах

Здесь нет никакой магии. Прошлая ошибка – не вина Ansible. Это не бэкап-система и она не умеет отказывать все к прошлым состояниям. Безопасность плейбуков – ваша ответственность. Ansible просто не знает как отменить эффект `a2ensite awesome-app`.

Как было сказано ранее, если задача не может исполнится – обработка останавливается... но мы можем принять ошибку (и нам [нужно это делать](http://www.aaronsw.com/weblog/geremiah)). Так мы и поступим: продолжим обработку в случае ошибки, но только чтобы вернуть все к рабочему состоянию.

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
          register: result
          ignore_errors: True

        - name: Rolling back - Restoring old default virtualhost
          command: a2ensite default
          when: result|failed

        - name: Rolling back - Removing our virtualhost
          command: a2dissite awesome-app
          when: result|failed

        - name: Rolling back - Ending playbook
          fail: msg="Configuration file is not valid. Please check that before re-running the playbook."
          when: result|failed

        - name: Deactivates the default virtualhost
          command: a2dissite default

        - name: Deactivates the default ssl virtualhost
          command: a2dissite default-ssl

        notify:
            - restart apache

      handlers:
        - name: restart apache
          service: name=apache2 state=restarted

Ключевое слово `register` записывает вывод команды `apache2ctl configtest` (exit 
status, stdout, stderr, ...), и `when: result|failed` проверяет содержит ли переменная 
(`result`) статус failed.

Поехали:

    $ ansible-playbook -i step-07/hosts -l host1.example.org step-07/apache.yml

    PLAY [web] ********************* 

    GATHERING FACTS ********************* 
    ok: [host1.example.org]

    TASK: [Installs apache web server] ********************* 
    ok: [host1.example.org]

    TASK: [Push future default virtual host configuration] ********************* 
    ok: [host1.example.org]

    TASK: [Activates our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Check that our config is valid] ********************* 
    failed: [host1.example.org] => {"changed": true, "cmd": ["apache2ctl", "configtest"], "delta": "0:00:00.051874", "end": "2013-03-10 10:50:17.714105", "rc": 1, "start": "2013-03-10 10:50:17.662231"}
    stderr: Syntax error on line 2 of /etc/apache2/sites-enabled/awesome-app:
    Invalid command 'RocumentDoot', perhaps misspelled or defined by a module not included in the server configuration
    stdout: Action 'configtest' failed.
    The Apache error log may have more information.
    ...ignoring

    TASK: [Rolling back - Restoring old default virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Removing our virtualhost] ********************* 
    changed: [host1.example.org]

    TASK: [Rolling back - Ending playbook] ********************* 
    failed: [host1.example.org] => {"failed": true}
    msg: Configuration file is not valid. Please check that before re-running the playbook.

    FATAL: all hosts have already failed -- aborting

    PLAY RECAP ********************* 
    host1.example.org              : ok=7    changed=4    unreachable=0    failed=1    

Кажется, все работает как нужно, Давайте попробуем перезапустить apache:

    $ ansible -i step-07/hosts -m service -a 'name=apache2 state=restarted' host1.example.org
    host1.example.org | success >> {
        "changed": true, 
        "name": "apache2", 
        "state": "started"
    }

Теперь наш Apache защищен от ошибок конфигурации.

Это может показаться большим объемом работы, но это не так. Помните, можно использовать переменные практически везде, так что можно использовать это как общий плейбук для apache в других случаях. Напишите один раз и используйте везде. Мы займемся этим в шаге 9, но пока давайте задеплоим сайт с помощью git в шаге [step-08](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-08).
