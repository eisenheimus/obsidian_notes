##### Настройка хостов Ansbile

##### /etc/ansible/hosts
```ini
[servers]
192.1.10.123
192.1.10.124 
```

##### Конфигурирование inventory
```
### Конфигурация inventory.ini

[servers]
192.1.10.123
192.1.10.124
```

 С указанием пользователя
```
[servers]
192.1.10.123 ansible_user=root
192.1.10.124 ansible_user=root
```

 С переменными для всей группы
```
[servers]
192.1.10.123
192.1.10.124  

[servers:vars]
ansible_user=root
ansible_python_interpreter=/usr/bin/python3
```

С разными пользователями для каждого хоста
```
[servers]
192.1.10.123 ansible_user=root
192.1.10.124 ansible_user=admin
```

С указанием порта (если не стандартный 22)
```ini
[servers]
192.1.10.123 ansible_user=root ansible_port=22
192.1.10.124 ansible_user=root ansible_port=2222
```

С указанием пути к приватному ключу
```
[servers]
192.1.10.123 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
192.1.10.124 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa

[servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

Полный вариант с алиасами
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