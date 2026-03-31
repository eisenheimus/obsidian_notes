```bash
# 3. Создаем базу и пользователя (замените 'strong_password' на ваш пароль)
sudo -u postgres psql <<EOF
CREATE USER planka WITH PASSWORD 'strong_password';
CREATE DATABASE planka OWNER planka;
GRANT ALL PRIVILEGES ON DATABASE planka TO planka;
```

или 

```bash
# Создание пользователя и базы данных
sudo -u postgres createuser --pwprompt planka
# Введите надежный пароль, например: Planka2024!Secure
sudo -u postgres createdb -O planka planka
# Настройка прав доступа
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE planka TO planka;"
```

https://chat.deepseek.com/a/chat/s/7a15b1b3-d29b-41ba-8e11-0ffb2c0533f1