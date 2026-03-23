

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
# Запуск контейнера
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

```bash
# Добавьте своего пользователя в группу docker
sudo usermod -aG docker $USER
```


```
sudo tee /etc/systemd/system/prometheus-docker.service > /dev/null << 'EOF'
[Unit]
Description=Prometheus Docker Container
After=docker.service
Requires=docker.service
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/docker start prometheus
ExecStop=/usr/bin/docker stop prometheus
ExecReload=/usr/bin/docker restart prometheus
Restart=on-failure
RestartSec=10
[Install]
WantedBy=multi-user.target
EOF
# 7. Запустите Prometheus вручную первый раз
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
# 8. Активируйте systemd сервис
sudo systemctl daemon-reload
sudo systemctl enable prometheus-docker.service
sudo systemctl start prometheus-docker.service
# 9. Проверьте
sudo systemctl status prometheus-docker.service
docker ps
curl localhost:9090/-/healthy
```