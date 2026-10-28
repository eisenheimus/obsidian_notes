https://habr.com/ru/articles/875548/
https://habr.com/ru/companies/lsfusion/articles/590599/

```bash
# обновляем списки пакетов
sudo apt update -y
```

```bash
# Загрузим PostgreSQL с утилитой postgresql-contrib
sudo apt install postgresql postgresql-contrib -y
```

```bash
# Добавляем автоазапуск
sudo systemctl enable --now postgresql
```

```bash
# Запустиьт сервис
sudo systemctl start postgresql.service
```


```bash
# Проверяем статус
sudo systemctl status postgresql.service
```


```bash
# переключимся на пользователя postgres
sudo -i -u postgres
```

```bash
# запускаем консоль SQL
psql
```

```sql
-- Создаем пользователя
postgres=#CREATE USER pg WITH PASSWORD '1234';
```

```sql
-- создаем БД
CREATE DATABASE test
```

```bash
# Показать все бд
\l
```

```sql
-- Выдать грант для пользователя pg на базу test
GRANT ALL PRIVILEGES ON DATABASE test TO test;
```

```bash
# Выходим из консоли pgsql
exit 
```

```bash
# Тестируем подключение к бд test и пользователем pg
psql test -U pg -h 127.0.0.1 -p 5432
```