```bash
# Запуск контейнеров в фоновом режиме
docker-compose up -d 
```

```bash
# Остановка и удаление контейнеров
docker-compose down 
```

```bash
# Просмотр логов PostgreSQL контейнера
docker-compose logs pgsql 
```

```bash
# Подключение к PostgreSQL внутри контейнера
docker-compose exec pgsql psql -U username -d [database]
```

```bash
# Просмотр статуса всех контейнеров
docker-compose ps 
```

```bash
# Перезапуск PostgreSQL контейнера
docker-compose restart pgsql 
```

```bash
# Вход в командную оболочку контейнера
docker-compose exec pgsql bash 
```

```bash
# Просмотр логов в реальном времени
docker-compose logs -f pgsql 
```

```bash
# Остановка только PostgreSQL контейнера
docker-compose stop pgsql 
```

```bash
# Запуск остановленного PostgreSQL контейнера
docker-compose start pgsql 
```

```bash
# Создание резервной копии базы данных
docker-compose exec pgsql pg_dump -U username database > backup.sql 
```

```bash
# Восстановление базы данных из резервной копии
docker-compose exec -T pgsql psql -U username database < backup.sql 
```

```bash
# Проверка готовности PostgreSQL к подключениям
docker-compose exec pgsql pg_isready -U [username]
```

```bash 
# Пересборка и запуск контейнера
docker-compose up -d --build pgsq
```

