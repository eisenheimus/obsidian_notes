### Работа с образами

```bash
# Список локальных образов
docker images

# Скачать образ из registry
docker pull <image>:<tag>

# Собрать образ из Dockerfile
docker build -t <name>:<tag> .

# Присвоить образу новый тег
docker tag <source> <target>

# Удалить образ
docker rmi <image>

# Принудительно удалить образ
docker rmi -f <image>
```



### Управление контейнерами (жизненный цикл)

```bash
# Запустить новый контейнер (создать + старт)
docker run <image>

# Запустить интерактивный контейнер с терминалом
docker run -it <image> /bin/bash

# Запустить контейнер в фоне (detached mode)
docker run -d <image>

# Запустить остановленный контейнер
docker start <container>

# Остановить запущенный контейнер
docker stop <container>

# Перезапустить контейнер
docker restart <container>

# Удалить остановленный контейнер
docker rm <container>

# Принудительно удалить (остановить + удалить)
docker rm -f <container>
```


### Мониторинг и диагностика

```bash
# Показать запущенные контейнеры
docker ps

# Показать все контейнеры (включая остановленные)
docker ps -a

# Посмотреть логи контейнера
docker logs <container>

# Следить за логами в реальном времени
docker logs -f <container>

# Зайти внутрь запущенного контейнера
docker exec -it <container> /bin/bash

# Инспектировать контейнер или образ (детальная информация)
docker inspect <container|image>

# Показать процессы внутри контейнера
docker top <container>

# Мониторинг ресурсов (CPU, RAM, сетевой трафик)
docker stats
```


### Очистка системы

```bash
# Удалить неиспользуемые контейнеры, сети, образы и кэш сборки
docker system prune

# Удалить всё неиспользуемое (включая все образы без контейнеров)
docker system prune -a

# Удалить все остановленные контейнеры
docker container prune

# Удалить "висячие" (dangling) образы
docker image prune
```


### Тома

```bash

# Список томов
docker volume ls

# Создать том
docker volume create <name>

# Пробросить директорию хоста в контейнер
docker run -v /host/path:/container/path ...

# Использовать именованный том
docker run -v <volume_name>:/container/path ...
```