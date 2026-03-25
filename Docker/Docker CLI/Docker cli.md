
```bash
# Показать запущенные контейнеры
docker pull [образ]:тег
```

```bash
# Показать запущенные контейнеры
docker ps
```

```bash
# Показать все контейнеры (включая остановленные)
docker ps -a
```

```bash
# Запустить новый контейнер из образа
docker run <image>
```

```bash
# Запустить интерактивный контейнер с терминалом
docker run -it <image> /bin/bash
```

```bash
# Запустить контейнер в фоне (detached mode)
docker run -d <image>
```

```bash
# Запустить остановленный контейнер
docker start <container>
```

```bash
# Остановить запущенный контейнер
docker stop <container>
```

```bash
# Перезапустить контейнер
docker restart <container>
```

```bash
# Удалить остановленный контейнер
docker rm <container>
```

```bash
# Принудительно удалить (остановить + удалить) контейнер
docker rm -f <container>
```

```bash
# Зайти внутрь запущенного контейнера
docker exec -it <container> /bin/bash
```

```bash
# Посмотреть логи контейнера
docker logs <container>
```

```bash
# Следить за логами в реальном времени (как tail -f)
docker logs -f <container>
```

```bash
# Список локальных образов
docker images
```

```bash
# Скачать образ из registry (например, docker pull nginx)
docker pull <image>
```

```bash
# Собрать образ из Dockerfile в текущей директории
docker build -t <name>:<tag> .
```

```bash
# Удалить образ
docker rmi <image>
```

```bash
# Принудительно удалить образ
docker rmi -f <image>
```

```bash
# Присвоить образу новый тег (для переименования или публикации)
docker tag <source> <target>
```

```bash
# Удалить неиспользуемые контейнеры, сети, образы и кэш сборки
docker system prune
```

```bash
# Удалить всё, что не используется, включая все образы без контейнеров
docker system prune -a
```

```bash
# Удалить все остановленные контейнеры
docker container prune
```

```bash
# Удалить "висячие" (dangling) образы
docker image prune
```

```bash
# Список сетей
docker network ls
```

```bash
# Список томов
docker volume ls
```

```bash
# Создать том
docker volume create <name>
```

```bash
# Пробросить директорию хоста в контейнер
docker run -v /host/path:/container/path ...
```

```bash
# Использовать именованный том
docker run -v <volume_name>:/container/path ...
```

```bash
# Инспектировать контейнер или образ
docker inspect <container|image>
```

```bash
# Показать процессы внутри контейнера
docker top <container>
```

```bash
# Мониторинг ресурсов (CPU, RAM, сетевой трафик) в реальном времени
docker stats
```

