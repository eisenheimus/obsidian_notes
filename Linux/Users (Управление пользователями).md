##### Частый кейс создания пользователя
```bash
# Cоздать пользователя Ubuntu/Debian
sudo adduser username

# Cоздать пользователя (все дистрибутивы)
# -m — создаёт домашнюю директорию (/home/username)
# -s /bin/bash — задаёт оболочку по умолчанию
sudo useradd -m -s /bin/bash username

# Задать пароль
sudo passwd username

# Добавит в группу sudo пользователя
sudo usermod -aG sudo username

# Проверить состоит ли в группе
groups username
```

```bash
# Создание пользователя с нестандарным каталогом
sudo useradd -m -d /var/www/webmaster -s /bin/bash webmaster
```

##### Cоздание пользователей
```bash
# Базовое создание пользователя
useradd username

# Создание с домашней директорией
useradd -m username

# Создание с указанием оболочки
useradd -m -s /bin/bash username

# Создание с указанием UID и комментария
useradd -m -s /bin/bash -u 1001 -c "Full Name" username

# Создание с указанием групп
useradd -m -g primarygroup -G secondarygroup1,secondarygroup2 username

# Создание с указанием срока действия
useradd -m -e 2024-12-31 username
```

##### Изменение существующих пользователей
```bash
# Изменение домашней директории
usermod -d /new/home/dir username

# Добавление в дополнительные группы
usermod -aG groupname username  # -a важно для добавления без удаления из других групп

# Изменение основной группы
usermod -g newprimarygroup username

# Изменение оболочки
usermod -s /bin/zsh username

# Блокировка/разблокировка пользователя
usermod -L username    # блокировка
usermod -U username    # разблокировка

# Изменение UID
usermod -u 2000 username

# Изменение срока действия
usermod -e 2025-12-31 username
```

##### Удаление пользователей
```bash
# Удаление пользователя
userdel username

# Удаление с домашней директорией и mail spool
userdel -r username
```

##### Полезные команды
```bash
# Смена пароля
passwd username

# Просмотр информации о пользователе
id username
finger username  # может потребоваться установка

# Просмотр активных пользователей
who
w
last

# Группы пользователя
groups username

# Смена владельца файлов
chown username:group file
chown -R username:group directory/
```

##### Практические примеры
```bash
# Создание пользователя со всем необходимым
sudo useradd -m -s /bin/bash -G sudo,www-data -c "John Doe" johndoe
sudo passwd johndoe

# Добавление существующего пользователя в группу docker
sudo usermod -aG docker username

# Изменение домашней директории и перемещение файлов
sudo usermod -d /new/home username
sudo mv /old/home/* /new/home/
sudo chown -R username:username /new/home

# Создание системного пользователя (без интерактивного входа)
sudo useradd -r -s /usr/sbin/nologin serviceuser
```

##### Важные файлы
```bash
# Файлы с информацией о пользователях
/etc/passwd     # информация о пользователях
/etc/shadow     # пароли (только root)
/etc/group      # информация о группах
/etc/sudoers    # права sudo (используйте visudo)
```
