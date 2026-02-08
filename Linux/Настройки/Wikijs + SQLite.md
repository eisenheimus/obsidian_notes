<br>

### 1. Установка Nodejs.

*Добавить  репозиторий nodesource*
```bash
curl -L https://deb.nodesource.com/setup_lts.x -o setup.sh
```
<br>

*Обновить пакеты*
```bash
sudo apt update -y
```
<br>

*Установить Nodejs*
```bash
sudo apt install nodejs -y
```
<br><br>








1 Создать пользователя wikiuser
su wikijs

2 wget https://github.com/Requarks/wiki/releases/latest/download/wiki-js.tar.gz
распаковать

	1. cp -a config.sample.yml confit.yml


создание демона
nano /etc/systemd/system/wikijs.service
---


[Unit] Description=Wiki.js After=network.target

[Service] Type=simple ExecStart=/usr/bin/node server Restart=always

User=wikijs Environment=NODE_ENV=production WorkingDirectory=/opt/wikijs

[Install] WantedBy=multi-user.target
---






















### 2. Установка и настройка SQLite

*Установка SQLite*
```bash
sudo apt update
sudo apt install sqlite3
```
<br>

*Создание базы данных SQLite*
```bash
# Создаем директорию для базы данных
sudo mkdir -p /var/lib/wikijs/db
sudo chown -R $USER:$USER /var/lib/wikijs

# Создаем базу данных
sqlite3 /var/lib/wikijs/db/wikijs.sqlite3 "VACUUM;"
```
<br><br>


### 3. Настройка конфигурации wikijs

```
скопировать конфиг
```

*Настройка конфигурации wikijs*
```bash
db:
  type: sqlite
  storage: /var/lib/wikijs/db/wikijs.sqlite3
```
<br>

*Установка драйвера SQLite для Node.js*
```bash
cd /path/to/wikijs
npm install sqlite3
```
<br>

*Настройка прав доступа*
```bash
sudo chown -R wikijs_user:wikijs_group /var/lib/wikijs/db
sudo chmod 755 /var/lib/wikijs/db
sudo chmod 644 /var/lib/wikijs/db/wikijs.sqlite3
```
<br>

*Перезапуск wikijs*
```bash
sudo systemctl restart wikijs

# или
cd /path/to/wikijs && npm start
```
<br>

*Backup базы данных*
```bash
cp /var/lib/wikijs/db/wikijs.sqlite3 /backup/wikijs_$(date +%Y%m%d).sqlite3
```