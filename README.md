# Подготовка к выполнению
- Установите Ansible версии 2.10 или выше.
 ```
 ansible --version
ansible [core 2.15.3]
  config file = None
  configured module search path = ['/home/devops/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.10/dist-packages/ansible
  ansible collection location = /home/devops/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.10.12 (main, Jun 11 2023, 05:26:28) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
```
- Создайте свой публичный репозиторий на GitHub с произвольным именем.
- Скачайте Playbook из репозитория с домашним заданием и перенесите его в свой репозиторий.
# Основная часть
- Попробуйте запустить playbook на окружении из test.yml, зафиксируйте значение, которое имеет факт some_fact для указанного хоста при выполнении playbook.
```
ansible -m ping localhost
[WARNING]: No inventory was parsed, only implicit localhost is available
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
ansible-playbook -i inventory/test.yml site.yml
PLAY [Print os facts] **************************************************************************************************
TASK [Gathering Facts] *************************************************************************************************ok: [localhost]
TASK [Print OS] ********************************************************************************************************ok: [localhost] => {
    "msg": "Ubuntu"
}
ansible-inventory -i inventory/test.yml --graph
@all:
  |--@ungrouped:
  |--@inside:
  |  |--localhost
devops@WORKBOOK:~/ansible/mnt-homeworks/08-ansible-01-base/playbook$ ansible-inventory -i inventory/prod.yml --graph
@all:
  |--@ungrouped:
  |--@el:
  |  |--centos7
  |--@deb:
  |  |--ubuntu

TASK [Print fact] ******************************************************************************************************ok: [localhost] => {
    "msg": 12
}
PLAY RECAP *************************************************************************************************************localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ansible-inventory -i inventory/prod.yml --list
{
    "_meta": {
        "hostvars": {
            "centos7": {
                "ansible_connection": "docker",
                "some_fact": "el"
            },
            "ubuntu": {
                "ansible_connection": "docker",
                "some_fact": "deb"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "el",
            "deb"
        ]
    },
    "deb": {
        "hosts": [
            "ubuntu"
        ]
    },
    "el": {
        "hosts": [
            "centos7"
        ]
    }
}
```
- Найдите файл с переменными (group_vars), в котором задаётся найденное в первом пункте значение, и поменяйте его на all default fact.
```
 ansible-playbook -i inventory/prod.yml site.yml

PLAY [Print os facts] **************************************************************************************************
TASK [Gathering Facts] *************************************************************************************************ok: [158.160.50.62]
ok: [84.201.172.212]

TASK [Print OS] ********************************************************************************************************ok: [158.160.50.62] => {
    "msg": "CentOS"
}
ok: [84.201.172.212] => {
    "msg": "Ubuntu"
}
TASK [Print fact] ******************************************************************************************************ok: [158.160.50.62] => {
    "msg": "el default fact"
}
ok: [84.201.172.212] => {
    "msg": "deb default fact"
}
PLAY RECAP *************************************************************************************************************158.160.50.62              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
84.201.172.212             : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
- Воспользуйтесь подготовленным (используется docker) или создайте собственное окружение для проведения дальнейших испытаний.
-  ВМ сзадала в YC
- Проведите запуск playbook на окружении из prod.yml. Зафиксируйте полученные значения some_fact для каждого из managed host.
- Добавьте факты в group_vars каждой из групп хостов так, чтобы для some_fact получились значения: для deb — deb default fact, для el — el default fact.
- Повторите запуск playbook на окружении prod.yml. Убедитесь, что выдаются корректные значения для всех хостов.
- При помощи ansible-vault зашифруйте факты в group_vars/deb и group_vars/el с паролем netology.
```
   ansible-vault encrypt group_vars/deb/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
devops@WORKBOOK:~/ansible/mnt-homeworks/08-ansible-01-base/playbook$ ansible-vault encrypt group_vars/el/examp.yml
New Vault password:
Confirm New Vault password:
Encryption successful
```
  ![vm](https://github.com/EVolgina/asible01/blob/main/exam.PNG)
- Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь в работоспособности.
```
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password:
PLAY [Print os facts] **************************************************************************************************
TASK [Gathering Facts] *************************************************************************************************ok: [158.160.50.62]
ok: [84.201.172.212]
TASK [Print OS] ********************************************************************************************************ok: [158.160.50.62] => {
    "msg": "CentOS"
}
ok: [84.201.172.212] => {
    "msg": "Ubuntu"
}
TASK [Print fact] ******************************************************************************************************ok: [158.160.50.62] => {
    "msg": "el default fact"
}
ok: [84.201.172.212] => {
    "msg": "deb default fact"
}
PLAY RECAP *************************************************************************************************************158.160.50.62              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
84.201.172.212             : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
- Посмотрите при помощи ansible-doc список плагинов для подключения. Выберите подходящий для работы на control node.
  ```
ansible-doc -t connection -l
  ```
- В prod.yml добавьте новую группу хостов с именем local, в ней разместите localhost с необходимым типом подключения.
- Запустите playbook на окружении prod.yml. При запуске ansible должен запросить у вас пароль. Убедитесь, что факты some_fact для каждого из хостов определены из верных group_vars.
```
ansible-playbook -i inventory/prod.yml site.yml --ask-vault-pass
Vault password:

PLAY [Print os facts] **************************************************************************************************************************************************
TASK [Gathering Facts] *************************************************************************************************************************************************ok: [158.160.50.62]
ok: [84.201.172.212]
ok: [127.0.0.1]

TASK [Print OS] ********************************************************************************************************************************************************ok: [158.160.50.62] => {
    "msg": "CentOS"
}
ok: [84.201.172.212] => {
    "msg": "Ubuntu"
}
ok: [127.0.0.1] => {
    "msg": "Ubuntu"
}

TASK [Print fact] ******************************************************************************************************************************************************ok: [158.160.50.62] => {
    "msg": "el default fact"
}
ok: [84.201.172.212] => {
    "msg": "deb default fact"
}
ok: [127.0.0.1] => {
    "msg": "local default fact"
}

PLAY RECAP *************************************************************************************************************************************************************
127.0.0.1                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
158.160.50.62              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
84.201.172.212             : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
- Заполните README.md ответами на вопросы. Сделайте git push в ветку master. В ответе отправьте ссылку на ваш открытый репозиторий с изменённым playbook и заполненным README.md.
