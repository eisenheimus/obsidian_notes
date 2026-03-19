### Bash

##### Запуск
```bash
# Подклчюиться к определнной БД
mysql -u root -p имя_базы_данных 
# или 
sudo -u root -p 

# Подключиться из консоли
mysql -h 192.168.1.100 -P 3307 -u root -p
```

##### Бэкапирование
```bash
# Бэкап базы
mysqldump -u root -p имя_базы > backup.sql

# Бэкап всех баз
mysqldump -u root -p --all-databases > all_backup.sql

# Восстановить базу
mysql -u root -p имя_базы < backup.sql
```

### SQL
##### БАЗЫ ДАННЫХ
```sql
-- Показать все базы
SHOW DATABASES;

-- Создать базу
CREATE DATABASE имя_базы;

-- Удалить базу (осторожно!)
DROP DATABASE имя_базы;

-- Выбрать базу для работы
USE имя_базы;

-- Показать текущую выбранную базу
SELECT DATABASE();
```

##### Таблицы
```sql
-- Показать все таблицы в текущей базе
SHOW TABLES;

-- Создать таблицу
CREATE TABLE пользователи (
    id INT AUTO_INCREMENT PRIMARY KEY,
    имя VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    возраст INT,
    дата_регистрации TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Показать структуру таблицы
DESCRIBE имя_таблицы;
-- или
DESC имя_таблицы;

-- Показать SQL-код создания таблицы
SHOW CREATE TABLE имя_таблицы;

-- Переименовать таблицу
RENAME TABLE старое_имя TO новое_имя;

-- Удалить таблицу
DROP TABLE имя_таблицы;

-- Очистить таблицу (удалить все строки)
TRUNCATE TABLE имя_таблицы;
```

##### Изменения таблиц
```sql
-- Добавить столбец
ALTER TABLE имя_таблицы ADD COLUMN новый_столбец VARCHAR(50);

-- Удалить столбец
ALTER TABLE имя_таблицы DROP COLUMN столбец;

-- Изменить тип столбца
ALTER TABLE имя_таблицы MODIFY COLUMN столбец TEXT;

-- Переименовать столбец
ALTER TABLE имя_таблицы CHANGE старое_имя новое_имя INT;

-- Добавить индекс
ALTER TABLE имя_таблицы ADD INDEX (столбец);

-- Добавить внешний ключ
ALTER TABLE имя_таблицы

ADD FOREIGN KEY (столбец) REFERENCES другая_таблица(ид);
```

##### Пользователи и права
```sql
-- Создать пользователя
CREATE USER 'имя'@'localhost' IDENTIFIED BY 'пароль';

-- Дать права
GRANT ALL PRIVILEGES ON база.* TO 'имя'@'localhost';
--или полные права на все БД и доступ извне
GRANT ALL PRIVILEGES ON *.* TO 'имя'@'%';

-- Показать права пользователя
SHOW GRANTS FOR 'имя'@'localhost';

-- Сменить пароль
ALTER USER 'имя'@'localhost' IDENTIFIED BY 'новый_пароль';

-- Удалить пользователя
DROP USER 'имя'@'localhost';

-- Применить изменения прав
FLUSH PRIVILEGES;
```

##### ИНФОРМАЦИЯ О СЕРВЕРЕ
```sql
-- Версия MySQL
SELECT VERSION();

-- Текущий пользователь
SELECT USER();

-- Все пользователи
SELECT user, host FROM mysql.user;

-- Статус сервера
STATUS;

-- Показать процессы
SHOW PROCESSLIST;

-- Показать переменные (с фильтром)
SHOW VARIABLES LIKE '%max%';
```





