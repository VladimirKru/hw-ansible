# Домашнее задание к занятию 2 «Работа с Playbook»

## Основная часть

1. Подготовьте свой inventory-файл `prod.yml`.
```markdown
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: 192.168.56.101
      ansible_user: vagrant
      ansible_port: 22
vector:
  hosts:
    vector-01:
      ansible_host: 192.168.56.102
      ansible_user: vagrant
      ansible_port: 22
```
2. Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает [vector](https://vector.dev). Конфигурация vector должна деплоиться через template файл jinja2.
```markdown
- name: Install Vector
  hosts: vector
  handlers:
      - name: Restart Vector
        become: true
        ansible.builtin.service:
          name: vector
          state: restarted
  tasks:
    - name: Get Vector Package
      become: true
      ansible.builtin.apt:
        deb: https://packages.timber.io/vector/0.30.0/vector_{{ vector_version }}-1_amd64.deb
        state: present
      notify: Restart Vector
```
3. При создании tasks рекомендую использовать модули: `get_url`, `template`, `unarchive`, `file`.
4. Tasks должны: скачать дистрибутив нужной версии, выполнить распаковку в выбранную директорию, установить vector.
5. Запустите `ansible-lint site.yml` и исправьте ошибки, если они есть.
```markdown
cio@hp-lx:~/devops-2023/git/hw-ansible/02-ansible/playbook$ ansible-lint site.yml
WARNING: PATH altered to include /usr/bin
WARNING  Overriding detected file kind 'yaml' with 'playbook' for given positional argument: site.yml
an AnsibleCollectionFinder has not been installed in this process
```
6. Попробуйте запустить playbook на этом окружении с флагом `--check`.
   ```markdown
    cio@hp-lx:~/devops-2023/git/hw-ansible/02-ansible/playbook$ ansible-playbook -i inventory/prod.yml site.yml --check

    PLAY [Install Clickhouse] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    Enter passphrase for key '/home/cio/.ssh/id_rsa': 
    ok: [clickhouse-01]

    TASK [Get clickhouse distrib] ***************************************************************************
    ok: [clickhouse-01] => (item=clickhouse-client)
    ok: [clickhouse-01] => (item=clickhouse-server)
    ok: [clickhouse-01] => (item=clickhouse-common-static)

    TASK [Install clickhouse packages] ***************************************************************************
    ok: [clickhouse-01] => (item=clickhouse-common-static-22.4.6.53.deb)
    ok: [clickhouse-01] => (item=clickhouse-client-22.4.6.53.deb)
    ok: [clickhouse-01] => (item=clickhouse-server-22.4.6.53.deb)

    TASK [Create database] ***************************************************************************
    skipping: [clickhouse-01]

    PLAY [Install Vector] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    ok: [vector-01]

    TASK [Get Vector Package] ***************************************************************************
    ok: [vector-01]

    PLAY RECAP ***************************************************************************
    clickhouse-01: ok=3    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
    vector-01: ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   ```
7. Запустите playbook на `prod.yml` окружении с флагом `--diff`. Убедитесь, что изменения на системе произведены.
   ```markdown
   cio@hp-lx:~/devops-2023/git/hw-ansible/02-ansible/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

    PLAY [Install Clickhouse] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    Enter passphrase for key '/home/cio/.ssh/id_rsa': 
    ok: [clickhouse-01]

    TASK [Get clickhouse distrib] ***************************************************************************
    changed: [clickhouse-01] => (item=clickhouse-client)
    changed: [clickhouse-01] => (item=clickhouse-server)
    changed: [clickhouse-01] => (item=clickhouse-common-static)

    TASK [Install clickhouse packages] ***************************************************************************
    Selecting previously unselected package clickhouse-common-static.
    (Reading database ... 26185 files and directories currently installed.)
    Preparing to unpack clickhouse-common-static-22.4.6.53.deb ...
    Unpacking clickhouse-common-static (22.4.6.53) ...
    Setting up clickhouse-common-static (22.4.6.53) ...
    changed: [clickhouse-01] => (item=clickhouse-common-static-22.4.6.53.deb)
    Selecting previously unselected package clickhouse-client.
    (Reading database ... 26199 files and directories currently installed.)
    Preparing to unpack clickhouse-client-22.4.6.53.deb ...
    Unpacking clickhouse-client (22.4.6.53) ...
    Setting up clickhouse-client (22.4.6.53) ...
    changed: [clickhouse-01] => (item=clickhouse-client-22.4.6.53.deb)
    Selecting previously unselected package clickhouse-server.
    (Reading database ... 26212 files and directories currently installed.)
    Preparing to unpack clickhouse-server-22.4.6.53.deb ...
    Unpacking clickhouse-server (22.4.6.53) ...
    Setting up clickhouse-server (22.4.6.53) ...
    changed: [clickhouse-01] => (item=clickhouse-server-22.4.6.53.deb)

    RUNNING HANDLER [Start clickhouse service] ***************************************************************************
    changed: [clickhouse-01]

    TASK [Create database] ***************************************************************************
    changed: [clickhouse-01]

    PLAY [Install Vector] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    ok: [vector-01]

    TASK [Get Vector Package] ***************************************************************************
    Selecting previously unselected package vector.
    (Reading database ... 26226 files and directories currently installed.)
    Preparing to unpack .../vector_0.30.0-1_amd64f9n8ztax.deb ...
    Unpacking vector (0.30.0-1) ...
    Setting up vector (0.30.0-1) ...
    systemd-journal:x:101:
    changed: [vector-01]

    PLAY RECAP ***************************************************************************
    clickhouse-01              : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    vector-01                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
   ```
8. Повторно запустите playbook с флагом `--diff` и убедитесь, что playbook идемпотентен.
   ```markdown
   cio@hp-lx:~/devops-2023/git/hw-ansible/02-ansible/playbook$ ansible-playbook -i inventory/prod.yml site.yml --diff

    PLAY [Install Clickhouse] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    Enter passphrase for key '/home/cio/.ssh/id_rsa': 
    ok: [clickhouse-01]

    TASK [Get clickhouse distrib] ***************************************************************************
    ok: [clickhouse-01] => (item=clickhouse-client)
    ok: [clickhouse-01] => (item=clickhouse-server)
    ok: [clickhouse-01] => (item=clickhouse-common-static)

    TASK [Install clickhouse packages] ***************************************************************************
    ok: [clickhouse-01] => (item=clickhouse-common-static-22.4.6.53.deb)
    ok: [clickhouse-01] => (item=clickhouse-client-22.4.6.53.deb)
    ok: [clickhouse-01] => (item=clickhouse-server-22.4.6.53.deb)

    TASK [Create database] ***************************************************************************
    ok: [clickhouse-01]

    PLAY [Install Vector] ***************************************************************************

    TASK [Gathering Facts] ***************************************************************************
    ok: [vector-01]

    TASK [Get Vector Package] ***************************************************************************
    ok: [vector-01]

    PLAY RECAP ***************************************************************************
    clickhouse-01              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    vector-01                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   ```
9.  Подготовьте README.md-файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги. П

### Playbook содержит два plays:

* Install Clickhouse
* Install Vector

### Clickhouse.

Группа хостов: 
Clickhouse
Состоит из 1 Hendler и 4 Tasks

В начале перезагужается сервер после его установки

#### Таски:

* Скачивает пакеты с сайта https://packages.clickhouse.com. Переменные (версия и тип дистрибутива) задаются в group_vars/clickhouse. Предусмотрен rescue сценарий для скачивания дистрибутива с альтернативного источника

* Устанавливает скаченные пакеты

* Запускает хендлер для перезапуска сервиса после установки

* Создание БД "logs". Определяет статусы: failed код возврата не 0 и changes если 0.

### Vector.
Группа хостов: vector

Состоит из 1 хендлера и 1 таски.

Хендлер нужен для перезапуска сервис.

Таска - устанавливает пакет, скачивая его с нужного ресурса по http. Переменная для указания версии определеяется в group_vars/vector.