
1. #### Обновление пакетов 
```bash
apt update -y
```

2. #### Установка зависимостей
```bash
apt install git curl wget
```

3. #### Установка [[Docker]]
4. #### Установка [[PostgreSQL]]
5. #### Скачиваем образ Planka
```bash
docker pull ghcr.io/plankanban/planka:latest
```

6. #### Создаем пользователя в SQl и выдаем права 
``` sql
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
```
7. #### Запуск Planka с официальным образом
```bash
docker run -d \
  --name planka-app \
  -p 3000:1337 \
  -e BASE_URL=http://localhost:3000 \
  -e DATABASE_URL=postgresql://admin:qwe123@1localhost:5432/test \
  -e SECRET_KEY="my_super_secret_key_123" \
  -v planka-uploads:/app/data \
  ghcr.io/plankanban/planka:latest
  
  # Или
  
  docker run -d \
  --name planka-app \
  -p 3000:1337 \
  -e BASE_URL=http://localhost:3000 \
  -e DATABASE_URL=postgresql://admin:qwe123@localhost:5432/test \
  -e SECRET_KEY="my_super_secret_key_123" \
  -e DEFAULT_ADMIN_EMAIL="ваш_email@example.com" \
  -e DEFAULT_ADMIN_PASSWORD="ваш_пароль" \
  -e DEFAULT_ADMIN_NAME="Administrator" \
  -e DEFAULT_ADMIN_USERNAME="admin" \
  -v planka-uploads:/app/data \
  ghcr.io/plankanban/planka:latest
```


```nginx
server {
    listen 80;
    server_name planka.yourdomain.com;
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

8. #### Добавляем в автозагрузку:
	8.1 через [[Docker автозапуск]]
	8.2 или unitfile
	
/etc/systemd/system/planka.service
```ini
[Unit]
Description=Planka Kanban Board
After=docker.service
Requires=docker.service
[Service]
Type=simple
ExecStart=/usr/bin/docker start -a planka-app
ExecStop=/usr/bin/docker stop -t 10 planka-app
ExecReload=/usr/bin/docker restart -t 10 planka-app
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

9. #### Активируйте сервис
```bash
sudo systemctl daemon-reload
sudo systemctl enable planka.service
sudo systemctl start planka.service
```

10. Проверьте
```bash
sudo systemctl status planka.service
docker ps | grep planka-app
```






























