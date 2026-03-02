sudo apt update
sudo apt install sqlite3

# Создаем директорию для базы данных
sudo mkdir -p /var/lib/wikijs/db
sudo chown -R $USER:$USER /var/lib/wikijs

# Создаем базу данных
sqlite3 /var/lib/wikijs/db/wikijs.sqlite3 "VACUUM;"

sudo nano /etc/wikijs/config.yml


```cfg
db:
  type: sqlite
  storage: /var/lib/wikijs/db/wikijs.sqlite3
```


cd /path/to/wikijs
npm install sqlite3

## 4. Установка драйвера SQLite для Node.js

cd /path/to/wikijs
npm install sqlite3

## 5. Настройка прав доступа

sudo chown -R wikijs_user:wikijs_group /var/lib/wikijs/db
sudo chmod 755 /var/lib/wikijs/db
sudo chmod 644 /var/lib/wikijs/db/wikijs.sqlite3

(Замените `wikijs_user` и `wikijs_group` на пользователя, под которым запускается wikijs)

## 6. Перезапуск wikijs

sudo systemctl restart wikijs
# или
cd /path/to/wikijs && npm start