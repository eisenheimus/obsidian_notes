```bash
# Создаем каталог
sudo mkdir -p /opt/planka/
```

```ini
# cоздаем /opt/planka/docker-compose.yaml

version: "3.8"

services:
  planka:
    image: ghcr.io/plankanban/planka:latest
    container_name: planka
    restart: unless-stopped
    ports:
      - "1337:1337"
    environment:
      - BASE_URL=${BASE_URL}
      - SECRET_KEY=${SECRET_KEY}
      - DATABASE_URL=${DATABASE_URL}
      - DEFAULT_ADMIN_EMAIL=${DEFAULT_ADMIN_EMAIL}
      - DEFAULT_ADMIN_PASSWORD=${DEFAULT_ADMIN_PASSWORD}
      - DEFAULT_ADMIN_NAME=${DEFAULT_ADMIN_NAME}
      - DEFAULT_ADMIN_USERNAME=${DEFAULT_ADMIN_USERNAME}
    volumes:
      - planka_uploads:/app/public/user-avatars
      - planka_attachments:/app/private/attachments
    depends_on:
      postgres:
        condition: service_healthy  # Ждать пока БД не станет здоровой
    networks:
      - planka_network

  postgres:
    image: postgres:16-alpine
    container_name: planka_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      #- POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - planka_network

volumes:
  planka_uploads:
    name: planka_uploads
  planka_attachments:
    name: planka_attachments
  postgres_data:
    name: planka_postgres_data

networks:
  planka_network:
    name: planka_network
    driver: bridge
```

```bash
# генерируем ключ (SECRET_KEY)
openssl rand -hex 64
```

```ini
# cоздаем /opt/planka/.env

# Planka Configuration
BASE_URL=http://localhost:1337
SECRET_KEY=key
DATABASE_URL=postgresql://postgres:postgres@postgres/planka

# Default Admin User
DEFAULT_ADMIN_EMAIL=admin@admin.com
DEFAULT_ADMIN_PASSWORD=secret_password
DEFAULT_ADMIN_NAME=Administrator
DEFAULT_ADMIN_USERNAME=admin

# Database Configuration
POSTGRES_DB=planka
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

```bash
# Запускаем контейнеры
dokcer compose up -d
```

```ini
# Добавляем а автозагрузку /etc/systed/system/planka.service

[Unit]
Description=Planka Docker Compose Service
Requires=docker.service
After=docker.service
[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/planka
# EnvironmentFile=/opt/planka/.env  ← МОЖНО УДАЛИТЬ
ExecStart=/usr/local/bin/docker-compose up -d
ExecStop=/usr/local/bin/docker-compose down
ExecReload=/usr/local/bin/docker-compose restart
User=root
StandardOutput=journal
[Install]
WantedBy=multi-user.target
```



```bash
# Проверить 

# Посмотрите IP адрес контейнера postgres в сети Docker
docker inspect planka_postgres | grep IPAddress

# Зайдите в контейнер planka и проверьте резолвинг имени
docker compose exec planka sh -c "ping -c 1 postgres"

# Или
docker compose exec planka sh -c "nslookup postgres"
```