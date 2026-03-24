

```bash
# Явное скачивание образа Prometheus
docker pull prom/prometheus:latest

# Или конкретной версии (рекомендуется для production)
docker pull prom/prometheus:v2.45.0
```

```bash
# Конфигурация в /etc
sudo mkdir -p /etc/prometheus
sudo chown -R 65534:65534 /etc/prometheus
```

```bash
# Данные в /var/lib (стандартное место для данных служб)
sudo mkdir -p /var/lib/prometheus
sudo chown -R 65534:65534 /var/lib/prometheus
```

```ini
# Создание конфигурации
sudo tee /etc/prometheus/prometheus.yml > /dev/null << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF
```

```bash
# Добавьте своего пользователя в группу docker
sudo usermod -aG docker $USER

# Применить группу docker
newgrp docke
```

```bash
# Запустите Prometheus вручную первый раз
docker run -d \
  --name prometheus \
  --restart unless-stopped \
  -p 9090:9090 \
  -v /etc/prometheus:/etc/prometheus \
  -v /var/lib/prometheus:/prometheus \
  prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/prometheus \
  --web.enable-lifecycle
```

```ini
# добавляем в автозагрузку /etc/systemd/system/prometheus-docker.service
[Unit]
Description=Prometheus Docker Container
Documentation=https://prometheus.io/docs/introduction/overview/
After=docker.service
Requires=docker.service
[Service]
Type=oneshot
RemainAfterExit=yes
User=root
Group=docker
# Запуск контейнера
ExecStart=/usr/bin/docker start prometheus
# Остановка контейнера
ExecStop=/usr/bin/docker stop prometheus
# Перезапуск контейнера
ExecReload=/usr/bin/docker restart prometheus
# Автоматический перезапуск при сбоях
Restart=on-failure
RestartSec=10
[Install]
WantedBy=multi-user.target

```

```bash
# Активируйте systemd сервис
sudo systemctl daemon-reload
sudo systemctl enable prometheus-docker.service
sudo systemctl start prometheus-docker.service
```

```bash
# Проверьте
sudo systemctl status prometheus-docker.service
docker ps
curl localhost:9090/-/healthy
```


