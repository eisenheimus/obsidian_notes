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
      # === КРИТИЧЕСКИ ВАЖНЫЕ НАСТРОЙКИ ДЛЯ WEBSOCKET ===
      - TRUST_PROXY=true
      - NODE_ENV=production
      - SAILS_CONFIG_SOCKETS_ONLYALLOWORIGINS=["http://localhost:1337"]
      - SAILS_SECURITY_CORS_ALLOW_ORIGINS=*
      - SAILS_SECURITY_CORS_ALLOW_ANY_ORIGIN_WITH_CREDENTIALS_UNSAFE=true
    volumes:
      - planka_uploads:/app/public/user-avatars
      - planka_attachments:/app/private/attachments
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - planka_network
  postgres:
    image: postgres:16-alpine
    container_name: planka_postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
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
SECRET_KEY=f9435012c3a4399904045ce05f33c49e020a411148d480517822f625f111914996dab1fae22f6ffa398a945a9af5680cec97bc512caaa7f2554a293d963a279d

# Default Admin User
DEFAULT_ADMIN_EMAIL=admin@admin.com
DEFAULT_ADMIN_PASSWORD=Liw1234est!
DEFAULT_ADMIN_NAME=Administrator
DEFAULT_ADMIN_USERNAME=admin

# Database Configuration
DATABASE_URL=postgresql://postgres:postgres@postgres/planka
POSTGRES_DB=planka
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

```bash
# Устанавливаем сгенерированный ключ
NEW_KEY=$(openssl rand -hex 64) && sudo sed -i "s/SECRET_KEY=.*/SECRET_KEY=$NEW_KEY/" .env
```

```bash
# Запускаем контейнеры
docker compose up -d
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
# Проверить сеть

# Посмотрите IP адрес контейнера postgres в сети Docker
docker inspect planka_postgres | grep IPAddress

# Зайдите в контейнер planka и проверьте резолвинг имени
docker compose exec planka sh -c "ping -c 1 postgres"

# Или
docker compose exec planka sh -c "nslookup postgres"
```


Траблшутинг

```bash
# Посмотреть логи planka
docker compose logs planka --tail=100

# Проверим подключение из контейнера Planka к PostgreSQL
docker exec -it planka sh -c "nc -zv postgres 5432"

# Остановить и удалить контейнеры и тома (важно!)
docker compose down -v
```
``
 Если проблема WS рабочий конфиг
 ```ini
 services:
  planka:
    image: lscr.io/linuxserver/planka:latest
    container_name: planka
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Moscow
      - BASE_URL=http://10.1.0.117:1337
      - SECRET_KEY=supersecretkeychangeit123456789
      - DATABASE_URL=postgresql://planka:plankapass@postgres/planka
      - DEFAULT_ADMIN_EMAIL=admin@admin.com
      - DEFAULT_ADMIN_PASSWORD=Liw1234est!
      - DEFAULT_ADMIN_NAME=Administrator
      - DEFAULT_ADMIN_USERNAME=admin
      # КРИТИЧЕСКИЕ НАСТРОЙКИ ДЛЯ WEBSOCKET
      - TRUST_PROXY=true
      - NODE_ENV=production
      - SAFE_CONNECT_ENV=1
      - SAILS_CONFIG_SOCKETS_ONLYALLOWORIGINS=["http://10.1.0.117:1337","http://localhost:1337"]
      - SAILS_CONFIG_LOG_LEVEL=verbose
    volumes:
      - planka_config:/config
    ports:
      - 1337:1337
    restart: unless-stopped
    depends_on:
      - postgres
  postgres:
    image: postgres:16-alpine
    container_name: planka_postgres
    environment:
      - POSTGRES_DB=planka
      - POSTGRES_USER=planka
      - POSTGRES_PASSWORD=plankapass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
volumes:
  planka_config:
  postgres_data:
 ```