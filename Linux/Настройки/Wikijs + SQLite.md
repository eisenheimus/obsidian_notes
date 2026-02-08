  <br>

# Установка Nodejs.

*Добавить  репозиторий nodesource*

```bash
curl -L https://deb.nodesource.com/setup_lts.x -o setup.sh
```

*Обновить пакеты*

```bash
sudo apt update -y
```

*Установить Nodejs*

```bash
sudo apt install nodejs -y
```
---
<br><br>

# Создание пользователя

*Создать пользователя wikiuser с каталогом в /home*

```bash
sudo useradd -m -s /bin/bash wikiuser
```

*Добавить пользователя в группу sudo*

```bash
sudo usermod -aG sudo wikiuser
```

*Назначить пароль (интерактивно)*

```bash
sudo userpasswd wikiuser
```
---
<br><br>

# Установка и настройка SQLite

*Обновляем пакеты*
```bash
sudo apt update
```

*Устанавливаем sqlite*
```bash
sudo apt install sqlite3
```

*Создаем директорию для базы данных*
```bash
sudo mkdir -p /var/lib/wikijs/db
```

*Назначаем владельца каталога*
```bash
sudo chown -R $USER:$USER /var/lib/wikijs
```

*Создаем базу данных*
```bash
sqlite3 /var/lib/wikijs/db/wikijs.sqlite3 "VACUUM;"
```
---
<br><br>

# Установка wikijs

*Меняем пользователя на wikijs*
```bash
su wikiuser
```

*Скачиваем wikijs*
```bash
wget https://github.com/Requarks/wiki/releases/latest/download/wiki-js.tar.gz
```

*Создаем каталог для wikijs*
```bash
sudo mkdir -p /opt/wikijs
```

*распаковываем*
```bash
sudo tar -xvf wiki-js.tar.gz /opt/wikijs
```
---
<br><br>

# Настройка конфигурации wikijs

*Создаем конфиг*
```bash
sudo cp -a config.sample.yml confit.yml
```

*Настройка конфигурации wikijs confit.yml*
```bash
db:
  type: sqlite
  storage: /var/lib/wikijs/db/wikijs.sqlite3
```

*Установка драйвера SQLite для Node.js*
```bash
cd /path/to/wikijs
npm install sqlite3
```

*Настройка прав доступа*
```bash
sudo chown -R wikijs_user:wikijs_group /var/lib/wikijs/db
sudo chmod 755 /var/lib/wikijs/db
sudo chmod 644 /var/lib/wikijs/db/wikijs.sqlite3
```

*Перезапуск wikijs*
```bash
sudo systemctl restart wikijs

# или
cd /path/to/wikijs && npm start
```

*Создаем демон /etc/systemd/system/wikijs.service*
```systemd
[Unit]
Description=Wiki.js
After=network.target


[Service]
Type=simple
ExecStart=/usr/bin/node server
Restart=always

User=wikijs
Environment=NODE_ENV=production
WorkingDirectory=/opt/wikijs

[Install]
WantedBy=multi-user.target
```

*Backup базы данных*
```bash
sudo cp /var/lib/wikijs/db/wikijs.sqlite3 /backup/wikijs_$(date +%Y%m%d).sqlite3
```