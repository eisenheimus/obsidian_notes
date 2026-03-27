```bash
# Обновляем
sudo apt update -y
```

```bash
# Устанавилваем зависимости
apt install -y wget
```

```bash
# Создаем пользователя trilium (без sudo прав)
# -r — создает системного пользователя (без домашней директории в /home)
# -m — создает домашнюю директорию
# -d /opt/trilium — указывает домашнюю директорию
# -s /bin/bash — устанавливает оболочку

sudo useradd -r -s /bin/bash -m -d /opt/trilium trilium
```

```bash
# Переключаемся на пользователя trilium
sudo -u trilium bash
```

```bash
wget https://github.com/TriliumNext/Trilium/releases/download/v0.102.1/TriliumNotes-Server-v0.102.1-linux-x64.tar.xz
```

```bash
# Создаем директорию и скачиваем
cd ~
wget https://github.com/TriliumNext/Trilium/releases/download/v0.102.1/TriliumNotes-Server-v0.102.1-linux-x64.tar.xz
```

```bash
# Распаковываем
tar -xf TriliumNotes-Server-v0.102.1-linux-x64.tar.xz
mv trilium-linux-x64-server trilium
cd trilium
```

```bash
# Делаем исполняемым
chmod +x trilium.sh
```

```bash
# завершаем сессию пользователя trilium
exit
```

/etc/systemd/system/trilium.service
```ini
[Unit]
Description=Trilium Notes
After=network.target

[Service]
Type=simple
User=trilium
Group=trilium
WorkingDirectory=/opt/trilium/trilium
ExecStart=/opt/trilium/trilium/trilium.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# Запускаем сервис

# Перезагружаем systemd
sudo systemctl daemon-reload

# Запускаем Trilium
sudo systemctl start trilium

# Добавляем в автозагрузку
sudo systemctl enable trilium

# Проверяем статус
sudo systemctl status trilium
```

```bash
# По умолчанию Trilium будет хранить данные в `/opt/trilium/trilium/data`. Убедитесь, что пользователь trilium имеет права на запись
sudo chown -R trilium:trilium /opt/trilium
```