<br>

```bash
# Cоздать пользователя Ubuntu/Debian
sudo adduser username
```
<br>


```bash
# Cоздать пользователя (все дистрибутивы)
sudo useradd -m -s /bin/bash username

# -m — создаёт домашнюю директорию (/home/username)
# -s /bin/bash — задаёт оболочку по умолчанию
```
<br>


```bash
# Задать пароль
sudo passwd username
```
<br>


```bash
# Добавит в группу sudo пользователя
sudo usermod -aG sudo username
```
<br>


```bash
# Проверить состоит ли в группе
groups username
```