Пособие по Ansible
================

Это пособие познакомит вас Ansible пошагово. Вам понадобится (виртуальная или реальная) машина, которая будет выступать в роли ansible node. Окружение для Vagrant идет в комплекте с этим пособием.

Ansible это программное решение для удаленного управления конфигурациями. Оно позволяет настраивать удаленные узлы. Главное его отличие от других подобных систем – Ansible использует (потенциально) существующую инфраструктуру SSH, в то время как другие (chef, puppet, ...) требует установки специального PKI-окружения.

Ansible использует т.н. push mode: конфигурация "проталкивается" (push) с главной машины. Другие CM-системы обычно поступают наоборот – узлы "тянут" (pull) конфигурацию с главной машины.

Этот режим интересен потому что вам не нужно иметь публично доступную главную машину для удаленной настройки узлов: это узлы должны быть доступны (позже мы увидим что скрытые узлы также могут получать конфигурацию), и большую часть времени они на самом деле доступны.

# Что нужно для Ansible

Необходимы следующие Python-модули
- python-yaml
- python-jinja2

На Debian/Ubuntu запустите:

``sudo apt-get install python-yaml python-jinja2 python-paramiko python-crypto``

У вас также должна быть комбинация ключей в ~/.ssh.

# Установка Ansible

## Из исходников

Ветка devel всегда стабильна, так что используем ее. Возможно, вам нужно будет установить git (`sudo apt-get install git` на Debian/Ubuntu).

    git clone git://github.com/ansible/ansible.git
    cd ./ansible

Теперь можно загрузить окружение Ansible.

    source ./hacking/env-setup

## Из deb пакета

    sudo apt-get install make fakeroot cdbs python-support
    git clone git://github.com/ansible/ansible.git
    cd ./ansible
    make deb
    sudo dpkg -i ../ansible_1.1_all.deb (version may vary)

В этом пособии мы допускаем, что вы использовали именно этот способ.

# Клонирование пособия

    git clone https://github.com/freetonik/ansible-tuto-rus.git
    cd ansible-tuto-rus

# Использование Vagrant в пособии

Настоятельно рекомендуем использовать Vagrant при прохождении этого пособия. Если вы еще не установили его, следуйте простым инструкциям в [step-00/README.md](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-00/README.md).

Если вы хотите продолжить без Vagnrant, переходите к шагу 
[step-01/README.md](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-01).

## Содержание

- [00. Установка Vagrant](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-00)
- [01. Basic inventory](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-01)
- [02. Первые модули и факты](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-02)
- [03. Группы и переменные](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-03)
- [04. Playbooks](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-04)
- [05. Плейбуки, отправка файлов на узлы](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-05)
- [06. Плейбуки и ошибки](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-06)
- [07. Плейбуки и условия](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-07)
- [08. Модуль Git](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-08)
- [09. Расширение до нескольких хостов](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-09)
- [10. Шаблоны](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-10)
- [11. Снова переменные](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-11)
- [12. Миграция к ролям](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-12)

## Contributing

Спасибо всем, кто участвовал в создании этого пособия:

* Aladin Jaermann
* Alexis Gallagher
* Atilla Mas
* Benny Wong
* Chris Schmitz
* dalton
* Daniel Howard
* David Golden
* Eugene Kalinin
* Hartmut Goebel
* Justin Garrison
* Karlo
* Marchenko Alexandr
* mxxcon
* Patrick Pelletier
* Pierre-Gilles Levallois
* Ruud Kamphuis
* Victor Boivie

Я использую Ansible почти с самого его появления, и я узнал очень много в процессе написания пособия. Если вы хотите поучаствовать – буду рад вашим дополнениям.

## От переводчика

Это перевод [туториала](https://github.com/leucos/ansible-tuto) от Michel Blanc. Разделы, работа над которыми идет в данный момент, находятся в ветке [writing](https://github.com/leucos/ansible-tuto/tree/writing) в оригинальном репозитории.

Если вы хотите дополнить/исправить перевод, пожалуйста, откройте Pull Request. 
