

https://chat.deepseek.com/a/chat/s/7a15b1b3-d29b-41ba-8e11-0ffb2c0533f1


```bash
# 1
apt update -y
```

```bash
# 2
apt install git curl wget
```

установка docker

скачиваем образ planka
docker pull ghcr.io/plankanban/planka:latest


# Запуск Planka с официальным образом
docker run -d \
  --name planka-app \
  -p 3000:1337 \
  -e BASE_URL=http://192.168.1.15:3000 \
  -e DATABASE_URL=postgresql://admin:qwe123@192.168.1.14:5432/test \
  -e SECRET_KEY="my_super_secret_key_123" \
  -v planka-uploads:/app/data \
  ghcr.io/plankanban/planka:latest





-- 1. Создание пользователя (пароль: ваш_надежный_пароль)
CREATE USER kanban_user WITH PASSWORD 'your_secure_password';
-- 2. Создание базы данных с владельцем kanban_user
CREATE DATABASE kanban OWNER kanban_user;
-- 3. Подключение к базе данных kanban
\c kanban;
-- 4. Передача прав на схему public пользователю kanban_user
GRANT ALL ON SCHEMA public TO kanban_user;
-- 5. Сделать kanban_user владельцем схемы public (опционально)
ALTER SCHEMA public OWNER TO kanban_user;
-- 6. Дать права на создание таблиц в схеме public
GRANT CREATE, USAGE ON SCHEMA public TO kanban_user;
-- 7. Дать все права на будущие таблицы в схеме public
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO kanban_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO kanban_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON FUNCTIONS TO kanban_user;

-- Создание пользователя
CREATE USER kanban_user WITH PASSWORD 'qwe123';
-- Создание базы данных с владельцем
CREATE DATABASE kanban OWNER kanban_user;
-- Подключение к новой базе
\c kanban
-- Права на схему public
GRANT ALL PRIVILEGES ON SCHEMA public TO kanban_user;
GRANT CREATE, USAGE ON SCHEMA public TO kanban_user;























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
# Обновление системы
apt update && apt upgrade -y

# Установка базовых зависимостей
apt install -y git curl wget

# Planka использует Sharp для работы с изображениями, требуются системные библиотеки
apt install -y \
  libgif-dev libglib2.0-dev liblcms2-dev libexif-dev \
  libgsf-bin libturbojpeg libpng-dev librsvg2-dev \
  libwebp-dev liborc-0.4-dev libsdl-pango-dev libtiff-dev \
  build-essential
```

### 2. установка nodejs + npm


### 3. Настройка PostgreSQL
```bash
# Создание пользователя и базы данных
sudo -u postgres createuser --pwprompt planka
# Введите надежный пароль, например: Planka2024!Secure

sudo -u postgres createdb -O planka planka

# Настройка прав доступа
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE planka TO planka;"
```
или
```bash
# 3. Создаем базу и пользователя (замените 'strong_password' на ваш пароль)
sudo -u postgres psql <<EOF
CREATE USER planka WITH PASSWORD 'strong_password';
CREATE DATABASE planka;
GRANT ALL PRIVILEGES ON DATABASE planka TO planka;
```

```bash
# Проверка подключения
# -h - хост
# -d - база данных
# -U - пользователь
psql -h 127.0.0.1 -d planka -U planka
```

### 4. Клонирование репозитория и установка
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

### 5. Настройка переменных окружения
```bash
cd /var/www/planka/server
cp .env.sample .env

# Генерация секретного ключа
SECRET_KEY=$(openssl rand -base64 64 | tr -dc _A-Z-a-z-0-9 | head -c 64)


# Отредактируйте файл .env:
nano /var/www/planka/server/.env
```

/var/www/planka/server/.env
```env
TZ=UTC
BASE_URL=http://IP_ВАШЕГО_КОНТЕЙНЕРА:1337
DATABASE_URL=postgresql://planka:ВАШ_ПАРОЛЬ@localhost:5432/planka
SECRET_KEY=ВАШ_СГЕНЕРИРОВАННЫЙ_КЛЮЧ
```

### 6.Инициализация базы данных
```bash
cd /var/www/planka
npm run server:db:init
```


### 7. Настройка systemd сервиса
```bash
# Создайте файл сервиса для автоматического запуска
nano /etc/systemd/system/planka.service
```

/etc/systemd/system/planka.service
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

```bash
# Установка прав
chown -R www-data:www-data /var/www/planka

# Включение сервиса
systemctl daemon-reload
systemctl enable planka
systemctl start planka

# Проверка статуса
systemctl status planka
```


### 8. Настройка Nginx
```bash
# Создайте конфигурацию Nginx
nano /etc/nginx/sites-available/planka
```

 /etc/nginx/sites-available/planka
```nginx
upstream planka_backend {
    server 127.0.0.1:1337;
    keepalive 32;
}

server {
    listen 80;
    server_name _;  # Замените на ваш домен или IP

    client_max_body_size 50M;

    location ~* \.io {
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://planka_backend;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://planka_backend;
        proxy_read_timeout 600s;
    }
}
```

```bash
# Активация сайта
ln -s /etc/nginx/sites-available/planka /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default  # Удаляем дефолтный сайт

# Проверка конфигурации
nginx -t

# Перезапуск Nginx
systemctl restart nginx
```

### 9. Доступ и первый вход
```bash
# Откройте браузер и перейдите по адресу http://IP_ВАШЕГО_КОНТЕЙНЕРА
# Учетные данные по умолчанию:
# Email: demo@demo.demo
# Пароль: demo
# Сразу смените пароль администратора после входа!
```

### 10. Настройка HTTPS (опционально).
```bash
# Установка certbot
apt install -y certbot python3-certbot-nginx

# Получение сертификата
certbot --nginx -d planka.yourdomain.com

# Автоматическое обновление
systemctl enable certbot.timer


# После получения сертификата обновите BASE_URL в .env на https://planka.yourdomain.com и перезапустите сервис
systemctl restart planka
```


### Полезные команды для управления
```bash
# Статус сервиса
systemctl status planka

# Просмотр логов
journalctl -u planka -f

# Перезапуск после изменений
systemctl restart planka

# Проверка логов Nginx
tail -f /var/log/nginx/planka-access.log
tail -f /var/log/nginx/planka-error.log
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