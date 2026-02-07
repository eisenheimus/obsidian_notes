
```text-x-trilium-auto
# установить OpenSSH
sudo apt install openssh-server -y
```

```text-x-trilium-auto
# Включим автозапуск
sudo systemctl enable ssh
```

```text-x-trilium-auto
# Проверим работоспособность
ssh localhost
```

```text-x-trilium-auto
# Правим конфиг
sudo nano /etc/ssh/sshd_config
```

```text-x-trilium-auto
Меняем в конфиге /etc/ssh/sshd_config:

Port 222

# отключить возможность входа на сервер учетной записи суперпользователя (root)

PermitRootLogin no

# добавить возможность входить через ключи

PubkeyAuthentication yes    

# отключения возможности входа по паролю 

PasswordAuthentication и присвоить no
```

```text-x-trilium-auto
# перезагрузить демон SSH
sudo systemctl restart ssh
```

```text-x-trilium-auto
# Переподключимся от обычной учетной записи и по другому порту
ssh -p 55555 ansible@95.213.154.235
```

```text-x-trilium-auto
# сгенерировать такую пару ключей
кssh-keygen -t rsa
```

# установить OpenSSH

sudo apt install openssh-server -y

  
 

# Включим автозапуск

sudo systemctl enable ssh

  
 

# Проверим работоспособность

ssh localhost

  
 

# Правим конфиг

sudo nano /etc/ssh/sshd_config  

  
 

Меняем:

Port 222

  
 

# отключить возможность входа на сервер учетной записи суперпользователя (root)

PermitRootLogin no

  
 

# добавить возможность входить через ключи

PubkeyAuthentication yes    

  
 

# отключения возможности входа по паролю 

PasswordAuthentication и присвоить no

  
  
 

# перезагрузить демон SSH

sudo systemctl restart ssh

  
  
 

# Переподключимся от обычной учетной записи и по другому порту

ssh -p 55555 ansible@95.213.154.235

  
  
  
  
 

# ssh-keygen -t rsa

  
 

#######################################################################################

  
  
  
  
  
  
 

# установить OpenSSH

sudo apt install openssh-server -y

  
 

# Включим автозапуск

sudo systemctl enable ssh

  
 

# Создать пользователя

# -m - создать домашний каталог

# -s - установит оболочку по умаолчанию

useradd -m -s /bin/bash  mariauser

  
 

# Задать пароль пльзователю интерактивно

passwd username

  
 

# добавить в группу sudo

usermod -aG sudo username

  
 

# Проверить группу ползователя

groups username 

  
  
  
 

ключ

  
 

# Если нет — создать новый (RSA 4096 или ed25519)

ssh-keygen -t ed25519 -C "your_email@example.com"

# или

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

  
  
  
  
  
 

# На локальной машине:

ssh-copy-id wikiuser@10.10.10.101

  
 

# Или вручную в контейнере:

mkdir -p /home/wikiuser/.ssh

chmod 700 /home/wikiuser/.ssh

echo "ваш_публичный_ключ" >> /home/wikiuser/.ssh/authorized_keys

chmod 600 /home/wikiuser/.ssh/authorized_keys

chown -R wikiuser:wikiuser /home/wikiuser/.ssh

  
  
 

# подключиться по ssh с конкретным ключом, если он не один в каталоге .ssh

ssh -i ~/.ssh/id_ed25519 -p 5000 wikiuser@10.10.10.101

  
  
 

# Натсроить безопасность для ssh в /etc/ssh/sshd_config

--------------------------------------------------------

# Отключить вход по паролю (только ключи!)

PasswordAuthentication no

  
 

# Запретить вход под root (если используете обычного пользователя)

PermitRootLogin no

# или

PermitRootLogin prohibit-password

  
 

# Перезагрузить SSH после изменений

systemctl reload ssh

  
  
 

Port 5000

--------------------------------------------------------

  
  
 

# Перезагрузить SSH после изменений

systemctl reload ssh