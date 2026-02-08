<br>

### 1. Генерация ключа на клиенте.

*Генерация ключа*
```bash
# RSA
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

#ED25519
ssh-keygen -t ed25519 -C "your_email@example.com"
```
<br><br>

### 2. Установка и запуск SSH на сервере  

*Установить OpenSSH*
```bash
sudo apt install openssh-server -y
```
<br>
    
*Включим автозапуск*
```bash
sudo systemctl enable ssh
```
<br>

*перезагрузить демон SSH*
```bash
sudo systemctl restart/start ssh
```
    

*Проверим работоспособность*
```bash
ssh localhost
```
<br><br>

### 3. Создаем пользователя

*Создать пользователя для ssh*
```bash
# -m - создать домашний каталог*
# -s - установит оболочку по умаолчанию

useradd -m -s /bin/bash sshuser
```
<br>

*Задать пароль пользователю интерактивно*
```bash
passwd sshuser
```
  

*добавить в группу sudo*
```bash
# -a - не удалять пользователя из других групп
# -G - назначить группу

usermod -aG sudo sshuser
```
  <br>

*Проверить группу пользователя*
```bash

groups sshuser

```
<br><br>

### 4. Сохраняем public key на сервере.  

*Вариант 1 (автоматический):*
```bash
# Исполняется на клиенте
ssh-copy-id wikiuser@10.10.10.10
```
  <br>

*Вариант 2 (ручной):*
```bash
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
<br><br>

### 5. Конфигурирование SSH
  
*Правим конфиг*
```bash
sudo nano /etc/ssh/sshd_config
```
  <br>

*Меняем в конфиге /etc/ssh/sshd_config:*
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
<br><br>

### 6. Подключение к серверу

```bash
# -i - путь к ключу ssh
# -p - порт

ssh -i ~/.ssh/id_ed25519 -p 5000 wikiuser@10.10.10.10
```