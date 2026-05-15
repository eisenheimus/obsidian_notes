#### Открываем порты в фаерволе

```bash
sudo ufw allow 9090/tcp comment 'Prometheus'
sudo ufw allow 3000/tcp comment 'Grafana'
sudo ufw allow 3100/tcp comment 'Loki'
sudo ufw reload
```


#### Создание структуры каталогов
```bash
# Создаем основную структуру
sudo mkdir -p /opt/monitoring/{data,logs,config}
cd /opt/monitoring
# Создаем подкаталоги для данных
mkdir -p data/{prometheus,grafana,loki}
mkdir -p logs/{prometheus,grafana,loki}
# Устанавливаем правильные права
sudo chown -R 472:472 data/grafana logs/grafana  # Grafana (uid 472)
# Prometheus работает от пользователя nobody (65534) в контейнере
# Но для совместимости с Docker Desktop иногда используют 1000:1000
sudo chown -R 65534:65534 data/prometheus logs/prometheus
sudo chown -R 65534:65534 data/loki logs/loki  # Loki (nobody)
sudo chmod 755 data logs
```

#### Создание конфигурационных файлов
##### /opt/monitoring/config/prometheus.yml
```yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'production'
alerting:
  alertmanagers:
    - static_configs:
        - targets: []
rule_files: []
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

##### /opt/monitoring/config/loki.yml

```yml
auth_enabled: false
server:
  http_listen_address: 0.0.0.0
  http_listen_port: 3100
  grpc_listen_address: 0.0.0.0
  grpc_listen_port: 9096
common:
  instance_addr: 0.0.0.0
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory
schema_config:
  configs:
    - from: 2024-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h
limits_config:
  allow_structured_metadata: true
  volume_enabled: true
  ingestion_rate_mb: 10
  ingestion_burst_size_mb: 20
analytics:
  reporting_enabled: false
compactor:
  working_directory: /loki/compactor
  compaction_interval: 10m
  retention_enabled: true
  retention_delete_delay: 2h
  retention_period: 744h  # 31 день
```


/opt/monitoring/docker-compose.yml
```yml
networks:
  monitoring:
    driver: bridge
    name: monitoring
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    networks:
      - monitoring
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./data/prometheus:/prometheus
      - ./logs/prometheus:/var/log/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--log.level=info'
      - '--web.enable-lifecycle'
    healthcheck:
      test: ["CMD", "wget", "-q", "--tries=1", "--spider", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    networks:
      - monitoring
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource
      - GF_SERVER_ROOT_URL=http://localhost:3000
      - GF_AUTH_ANONYMOUS_ENABLED=false
    volumes:
      - ./data/grafana:/var/lib/grafana
      - ./logs/grafana:/var/log/grafana
    depends_on:
      prometheus:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"
  loki:
    image: grafana/loki:latest
    container_name: loki
    restart: unless-stopped
    networks:
      - monitoring
    ports:
      - "3100:3100"  # Порт для приема логов с удаленных машин
      - "9096:9096"  # GRPC порт
    volumes:
      - ./config/loki.yml:/etc/loki/loki-config.yaml:ro
      - ./data/loki:/loki
      - ./logs/loki:/var/log/loki
    command: -config.file=/etc/loki/loki-config.yaml
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3100/ready"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"
volumes:
  prometheus_data:
    driver: local
  grafana_data:
    driver: local
  loki_data:
    driver: local
```


/opt/monitoring/.env
```ini
# Grafana credentials
GRAFANA_USER=admin
GRAFANA_PASSWORD=qwe123
# Docker Compose project name
COMPOSE_PROJECT_NAME=monitoring
# Loki settings (если нужна авторизация)
# LOKI_AUTH_ENABLED=false
```

```bash
# устанавливаем права на файл
chmod 600 /opt/monitoring/.env
```


#### Проверка
```bash
# Проверяем конфигурацию
docker compose config
# Запускаем сервисы
docker compose up -d
# Проверяем статус
docker compose ps
# Смотрим логи на предмет ошибок
docker compose logs --tail=50
# Проверяем доступность
curl http://localhost:9090  # Prometheus
curl http://localhost:3000  # Grafana
curl http://localhost:3100/ready  # Loki


# Проверка логов на ошибки
docker compose logs --tail=50 | grep -i error
# Проверка что все эндпоинты работают
curl -I http://localhost:9090/api/v1/query?query=up
curl -I http://localhost:3100/ready  
curl -I http://localhost:3000/api/health
```


#### Настройка systemd для автозапуска
```ini
# Создаем systemd сервис
sudo tee /etc/systemd/system/monitoring.service << 'EOF'
[Unit]
Description=Monitoring Stack (Prometheus + Grafana + Loki)
Requires=docker.service network-online.target
After=docker.service network-online.target
StartLimitIntervalSec=0
StartLimitBurst=5
[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/monitoring
EnvironmentFile=/opt/monitoring/.env
ExecStartPre=/usr/bin/docker compose pull
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
ExecReload=/usr/bin/docker compose restart
Restart=no
StandardOutput=journal
StandardError=journal
[Install]
WantedBy=multi-user.target
EOF
# Активируем сервис
sudo systemctl daemon-reload
sudo systemctl enable monitoring.service
sudo systemctl start monitoring.service
# Проверяем статус
sudo systemctl status monitoring.service
```


#### Для production заменить `latest` на конкретные версии:
```
image: prom/prometheus:v2.53.0
image: grafana/grafana:11.1.0
image: grafana/loki:3.1.0
```


#### Добавить резервное копирование данных (crontab)
```
0 2 * * * tar -czf /backup/monitoring-$(date +\%Y\%m\%d).tar.gz /opt/monitoring/data/
```