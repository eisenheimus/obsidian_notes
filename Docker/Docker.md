<br>

```bash
# Показать запущенные контейнеры
docker ps
```
<br>


```bash
# Показать все контейнеры (включая остановленные)
docker ps -a
```
<br>


```bash
# Запустить новый контейнер из образа
docker run <image>
```
<br>


```bash
# Запустить интерактивный контейнер с терминалом
docker run -it <image> /bin/bash
```
<br>


```bash
# Запустить контейнер в фоне (detached mode)
docker run -d <image>
```
<br>


```bash
# Запустить остановленный контейнер
docker start <container>
```
<br>


```bash
# Остановить запущенный контейнер
docker stop <container>
```
<br>


```bash
# Перезапустить контейнер
docker restart <container>
```
<br>


```bash
# Удалить остановленный контейнер
docker rm <container>
```
<br>


```bash
# Принудительно удалить (остановить + удалить) контейнер
docker rm -f <container>
```
<br>


```bash
# Зайти внутрь запущенного контейнера
docker exec -it <container> /bin/bash
```
<br>


```bash
# Посмотреть логи контейнера
docker logs <container>
```
<br>


```bash
# Следить за логами в реальном времени (как tail -f)
docker logs -f <container>
```
<br>


```bash
# Список локальных образов
docker images
```
<br>


```bash
# Скачать образ из registry (например, docker pull nginx)
docker pull <image>
```
<br>


```bash
# Собрать образ из Dockerfile в текущей директории
docker build -t <name>:<tag> .
```
<br>


```bash
# Удалить образ
docker rmi <image>
```
<br>


```bash
# Принудительно удалить образ
docker rmi -f <image>
```
<br>


```bash
# Присвоить образу новый тег (для переименования или публикации)
docker tag <source> <target>
```
<br>


```bash
# Удалить неиспользуемые контейнеры, сети, образы и кэш сборки
docker system prune
```
<br>


```bash
# Удалить всё, что не используется, включая все образы без контейнеров
docker system prune -a
```
<br>


```bash
# Удалить все остановленные контейнеры
docker container prune
```
<br>


```bash
# Удалить "висячие" (dangling) образы
docker image prune
```
<br>


```bash
# Список сетей
docker network ls
```
<br>


```bash
# Список томов
docker volume ls
```
<br>


```bash
# Создать том
docker volume create <name>
```
<br>


```bash
# Пробросить директорию хоста в контейнер
docker run -v /host/path:/container/path ...
```
<br>

```bash
# Использовать именованный том
docker run -v <volume_name>:/container/path ...
```
<br>


```bash
# Инспектировать контейнер или образ
docker inspect <container|image>
```
<br>


```bash
# Показать процессы внутри контейнера
docker top <container>
```
<br>


```bash
# Мониторинг ресурсов (CPU, RAM, сетевой трафик) в реальном времени
docker stats
```
<br>

