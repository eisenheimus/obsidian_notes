<br>

# Генерация ключа на клиенте.

```bash
# Генерация ключа RSA
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Генерация ключа ED25519
ssh-keygen -t ed25519 -C "your_email@example.com"
```
---
<br>

# Установка и запуск SSH на сервере  


```bash
# Установить OpenSSH
sudo apt install openssh-server -y
```
<br>

```bash
# Включим автозапуск
sudo systemctl enable ssh
```
<br>

```bash
# перезагрузить демон SSH
sudo systemctl restart/start ssh
```
<br>

```bash
# Проверим работоспособность
ssh localhost
```
---
<br>

# Создаем пользователя

```bash
# Создать пользователя для ssh
useradd -m -s /bin/bash sshuser

# -m - создать домашний каталог
# -s - установит оболочку по умаолчанию
```
<br>

```bash
# Задать пароль пользователю интерактивно
passwd sshuser
```
  <br>

```bash
# добавить в группу sudo
usermod -aG sudo sshuser

# -a - не удалять пользователя из других групп
# -G - назначить группу
```
<br>

```bash
# Проверить группу пользователя
groups sshuser

```
---
<br>

# Сохраняем public key на сервере.  

```bash
# Вариант 1 (автоматический):
# Исполняется на клиенте
ssh-copy-id wikiuser@10.10.10.10
```
<br>

```bash
# Вариант 2 (ручной):
# Создать каталог .ssh
mkdir -p /home/user/.ssh

# Назначить полный доступ для владельца
chmod 700 /home/wikiuser/.ssh

# Назначить владельца
chown -R user:user /home/user/.ssh

# Созхранить публичный ключ
echo "ваш_публичный_ключ" >> /home/wikiuser/.ssh/authorized_keys

# Назначить доступ на чтение и запись для владельца
chmod 600 /home/wikiuser/.ssh/authorized_keys
```
---
<br>

# Конфигурирование SSH
  
```bash
# Правим конфиг
sudo nano /etc/ssh/sshd_config
```
<br>

##### /etc/ssh/sshd_config:
```bash
# Устанавливаем кастомный порт
Port 222

#отключить возможность входа на сервер учетной записи суперпользователя (root)
PermitRootLogin no

#добавить возможность входить через ключи
PubkeyAuthentication yes    

#отключения возможности входа по паролю
PasswordAuthentication и присвоить no
```
---
<br>

# Подключение к серверу
```bash
ssh -i ~/.ssh/id_ed25519 -p 5000 wikiuser@10.10.10.10

# -i - путь к ключу ssh
# -p - порт
```

# Если вдруг 22 порт не закрывается а новый порт не поднимается. 
```bash
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
```