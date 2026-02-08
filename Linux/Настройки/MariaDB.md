<br>

# Установка 

```bash
# Обновление пакетов
sudo apt update
```
<br>


```bash
# Установка сервера MariaDB
sudo apt install -y mariadb-server
```
<br>

```bash
# Просмотр статуса
sudo systemctl status mariadb
```
---
<br>




# Создать креды для БД

```bash
# Запуск консоли MySQL
sudo mysql
```
<br>


```sql
-- Создание пользователя (локальный доступ)
CREATE USER 'mariauser'@'localhost' IDENTIFIED BY 'ваш_надёжный_пароль';
 
-- Для удалённого доступа (если нужно):
CREATE USER 'mariauser'@'%' IDENTIFIED BY 'ваш_надёжный_пароль';

-- Выдача прав (пример: полный доступ к БД mydb)
CREATE DATABASE IF NOT EXISTS mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

GRANT ALL PRIVILEGES ON mydb.* TO 'mariauser'@'localhost';

-- Применение изменений
FLUSH PRIVILEGES;

EXIT;
```
---
<br>

# Проверка подключения


```bash
mysql -u mariauser -p -D mydb

# или 
sudo mysql -u root -p
```
---
<br>

# Настройка безопасности в БД
```SQL
-- Установить пароль для root (если еще нет)
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Ваш_надежный_пароль_123!';

-- Удалить анонимных пользователей
DELETE FROM mysql.user WHERE User='';

-- Запретить удаленный вход root
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');

-- Удалить тестовую базу данных
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';

-- Обновить привилегии
FLUSH PRIVILEGES;

-- Выйти
EXIT;
```
---
<br>