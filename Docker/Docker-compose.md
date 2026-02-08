```bash
# Запуск контейнеров в фоновом режиме
docker-compose up -d 
```
<br>


```bash
# Остановка и удаление контейнеров
docker-compose down 
```
<br>


```bash
# Просмотр логов PostgreSQL контейнера
docker-compose logs pgsql 
```
<br>


```bash
# Подключение к PostgreSQL внутри контейнера
docker-compose exec pgsql psql -U username -d [database]
```
<br>


```bash
# Просмотр статуса всех контейнеров
docker-compose ps 
```
<br>


```bash
# Перезапуск PostgreSQL контейнера
docker-compose restart pgsql 
```
<br>


```bash
# Вход в командную оболочку контейнера
docker-compose exec pgsql bash 
```
<br>


```bash
# Просмотр логов в реальном времени
docker-compose logs -f pgsql 
```
<br>


```bash
# Остановка только PostgreSQL контейнера
docker-compose stop pgsql 
```
<br>


```bash
# Запуск остановленного PostgreSQL контейнера
docker-compose start pgsql 
```
<br>


```bash
# Создание резервной копии базы данных
docker-compose exec pgsql pg_dump -U username database > backup.sql 
```
<br>


```bash
# Восстановление базы данных из резервной копии
docker-compose exec -T pgsql psql -U username database < backup.sql 
```
<br>


```bash
# Проверка готовности PostgreSQL к подключениям
docker-compose exec pgsql pg_isready -U [username]
```
<br>


```bash 
# Пересборка и запуск контейнера
docker-compose up -d --build pgsq
```

