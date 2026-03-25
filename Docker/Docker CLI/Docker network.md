```bash
# Проверка IP контейнеров в пользовательской сети
docker network inspect monitoring
```

```bash
# Создаём пользовательскую сеть
docker network create monitoring
```

```bash
# Запуск контейнера с указанием сети
docker run -d --name prometheus --network monitoring ...
```

```bash
# Или подключение существующего
docker network connect monitoring prometheus
```

```bash
# Из контейнера grafana проверим резолвинг имени prometheus
docker exec grafana ping prometheus

# Или через nslookup
docker exec grafana nslookup prometheus
```