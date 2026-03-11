
```bash
# Cоздать пользователя Ubuntu/Debian
sudo adduser username
```


```bash
# Cоздать пользователя (все дистрибутивы)
sudo useradd -m -s /bin/bash username

# -m — создаёт домашнюю директорию (/home/username)
# -s /bin/bash — задаёт оболочку по умолчанию
```


```bash
# Задать пароль
sudo passwd username
```


```bash
# Добавит в группу sudo пользователя
sudo usermod -aG sudo username
```


```bash
# Проверить состоит ли в группе
groups username
```