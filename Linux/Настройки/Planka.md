

https://chat.deepseek.com/a/chat/s/7a15b1b3-d29b-41ba-8e11-0ffb2c0533f1


```bash
# 1
apt update -y
```

```bash
# 2
apt install git curl wget
```

3. Установка [[Node.js]] + npm

```bash
# зависимости planka
apt install -y \
  libgif-dev libglib2.0-dev liblcms2-dev libexif-dev \
  libgsf-bin libturbojpeg libpng-dev librsvg2-dev \
  libwebp-dev liborc-0.4-dev libsdl-pango-dev libtiff-dev \
  build-essential
```


```bash
# Создание директории
mkdir -p /var/www/planka
cd /var/www/planka

# Клонирование репозитория
git clone https://github.com/plankanban/planka.git .

# Установка зависимостей
npm install

# Сборка клиентской части
npm run client:build

# Копирование собранного клиента в директорию сервера
cp -r client/build/. server/public
cp client/build/index.html server/views/index.ejs
```



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



```
cd /var/www/planka/server
cp .env.sample .env
# Генерация секретного ключа
SECRET_KEY=$(openssl rand -base64 64 | tr -dc _A-Z-a-z-0-9 | head -c 64)
```

```
nano /var/www/planka/server/.env
```

```
TZ=UTC
BASE_URL=http://IP_ВАШЕГО_КОНТЕЙНЕРА:1337
DATABASE_URL=postgresql://planka:ВАШ_ПАРОЛЬ@localhost:5432/planka
SECRET_KEY=ВАШ_СГЕНЕРИРОВАННЫЙ_КЛЮЧ
```


```
cd /var/www/planka
npm run server:db:init
```


```ini
[Unit]
Description=Planka Kanban Board
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/var/www/planka/server
Environment=NODE_ENV=production
ExecStart=/usr/bin/npm start
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```
# Установка прав
chown -R www-data:www-data /var/www/planka
# Включение сервиса
systemctl daemon-reload
systemctl enable planka
systemctl start planka
# Проверка статуса
systemctl status planka
```