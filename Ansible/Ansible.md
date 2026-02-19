### Установка

```bash

# Установить пакеты
sudo yum update

# добавьте репозиторий EPEL (Extra Packages for Enterprise Linux)
sudo yum install epel-release

# Затем установите Ansible
sudo yum install ansible

# После установки Ansible убедитесь, что установка прошла успешно
ansible --version

```


### Настройка хостов Ansbile

##### /etc/ansible/hosts
```ini
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com

[servers]
192.1.10.123
192.1.10.124 
```

``` bash
# Настройте аутентификацию на основе ключей SSH
ssh-keygen

# Скопируйте открытый ключ на управляемые узлы.
ssh-copy-id root@192.1.10.123
ssh-copy-id root@192.1.10.124

# Проверьте SSH-соединение
ssh root@192.1.10.123
ssh root@192.1.10.124

# Проверьте, может ли Ansible подключаться к управляемым хостам
ansible -m ping all
```



```
# Проверьте, может ли Ansible подключаться к управляемым хостам.
ansible -m ping all

```
### Запуск

```bash
# Запуск плейбука с использованием инвентаря по умолчанию (обычно /etc/ansible/hosts)
ansible-playbook playbook.yml

# Запуск плейбука с указанием конкретного файла инвентаря в формате INI
ansible-playbook -i inventory.ini playbook.yml

# Запуск плейбука с использованием файла "hosts" в качестве инвентаря
ansible-playbook -i hosts playbook.yml

# Запуск плейбука только для одного хоста "webserver1" из инвентаря
ansible-playbook -i inventory.ini playbook.yml --limit webserver1

# Запуск плейбука в режиме проверки (dry-run) без фактического внесения изменений
ansible-playbook playbook.yml --check

# Запуск плейбука с подробным выводом (уровень детализации - один "v")
ansible-playbook playbook.yml -v

# Запуск плейбука с запросом пароля для подключения по SSH (ask-pass)
ansible-playbook playbook.yml -k

# Запуск плейбука с запросом пароля sudo для повышения привилегий (ask-become-pass)
ansible-playbook playbook.yml -K

# Запуск плейбука с запросом обоих паролей: для SSH и для sudo
ansible-playbook playbook.yml -k -K

# Запуск плейбука с запросом пароля для расшифровки зашифрованных файлов Ansible Vault
ansible-playbook playbook.yml --ask-vault-pass

# Запуск плейбука с использованием файла, содержащего пароль для расшифровки Ansible Vault
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass.txt
```


### Конфигурация inventory.ini

##### Минимальный вариант (только список хостов)
```
[servers]
192.1.10.123
192.1.10.124
```

##### С указанием пользователя
```
[servers]
192.1.10.123 ansible_user=root
192.1.10.124 ansible_user=root
```

##### С переменными для всей группы
```
[servers]
192.1.10.123
192.1.10.124  

[servers:vars]
ansible_user=root
ansible_python_interpreter=/usr/bin/python3
```

##### С разными пользователями для каждого хоста
```
[servers]
192.1.10.123 ansible_user=root
192.1.10.124 ansible_user=admin
```

##### С указанием порта (если не стандартный 22)
```ini
[servers]
192.1.10.123 ansible_user=root ansible_port=22
192.1.10.124 ansible_user=root ansible_port=2222
```

##### С указанием пути к приватному ключу
```
[servers]
192.1.10.123 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
192.1.10.124 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

##### Полный вариант с алиасами
```
[servers]
server1 ansible_host=192.1.10.123 ansible_user=root
server2 ansible_host=192.1.10.124 ansible_user=root

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_timeout=30
```

##### Примеры запуска
```
# Проверить подключение к группе
ansible servers -m ping -i inventory.ini

# Проверить конкретный хост
ansible 192.1.10.123 -m ping -i inventory.ini

# Запустить плейбук
ansible-playbook -i inventory.ini playbook.yml
```