https://www.youtube.com/watch?v=FePECRTCzBs
  <br>

# Установка Nodejs.

```bash
# Добавить  репозиторий nodesource
curl -L https://deb.nodesource.com/setup_lts.x -o setup.sh
```
<br>

```bash
# Обновить пакеты
sudo apt update -y
```
<br>

```bash
# Установить Nodejs
sudo apt install nodejs npm -y 
```
---
<br>

# Создание пользователя

```bash
# Создать пользователя wikiuser с каталогом в /home
sudo useradd -m -s /bin/bash wikiuser
```
<br>

```bash
# Добавить пользователя в группу sudo
sudo usermod -aG sudo wikiuser
```
<br>

```bash
# Назначить пароль (интерактивно)
sudo userpasswd wikiuser
```
---
<br>

# Установка wikijs

```bash
# Меняем пользователя на wikijs
su wikiuser
```
<br>

```bash
# Скачиваем wikijs
wget https://github.com/Requarks/wiki/releases/latest/download/wiki-js.tar.gz
```
<br>

```bash
# Создаем каталог для wikijs
sudo mkdir -p /opt/wikijs
```
<br>

```bash
# распаковываем
sudo tar -xvf wiki-js.tar.gz -C /opt/wikijs
```
---
<br>


# Установка и настройка SQLite

```bash
# Обновляем пакеты
sudo apt update
```
<br>

```bash
# Устанавливаем sqlite
sudo apt install sqlite3
```
<br>

```bash
# Создаем директорию для базы данных
sudo mkdir -p /var/lib/wikijs/db
```
<br>

```bash
# Назначаем владельца каталога
sudo chown -R $USER:$USER /var/lib/wikijs
```
<br>

```bash
# Создаем базу данных
sqlite3 /var/lib/wikijs/db/wikijs.sqlite3 ""

# Важно указать пустые кавычки в конце "" иначе файл бд не создастя
```
<br>

```bash
# Установка драйвера SQLite для Node.js
cd /opt/wikijs
npm install --save sqlite3

#или force в случае конфлитка
npm install --save --force sqlite3 && npm audit fix --force 
```
---
<br>


# Настройка конфигурации wikijs

```bash
# Создаем конфиг
sudo cp -a config.sample.yml config.yml
```
<br>

```bash
# Настройка конфигурации wikijs confit.yml
db:
  type: sqlite
  storage: /var/lib/wikijs/db/wikijs.sqlite3
```
<br>



```bash
# Настройка прав доступа
sudo chown -R wikijs_user:wikijs_group /var/lib/wikijs/db
sudo chmod 755 /var/lib/wikijs/db
sudo chmod 644 /var/lib/wikijs/db/wikijs.sqlite3
```
<br>



# Создаем демон /etc/systemd/system/wikijs.service
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
<br>

```bash
# Перечитать конфиги systemd
sudo systemctl daemon-reload

# Включить автозапуск
sudo systemctl enable wikijs

# Перезапуск wikijs
sudo systemctl restart wikijs

# Посмотреть логи
journalctl -u wikijs -f
```
<br>

```bash
# Backup базы данных
sudo cp /var/lib/wikijs/db/wikijs.sqlite3 /backup/wikijs_$(date +%Y%m%d).sqlite3
```