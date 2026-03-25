

> Docker layers (слои Docker) — это ключевая концепция в Docker, которая помогает эффективно управлять контейнерами и их образом. Docker образ состоит из цепочки слоев, где каждый слой представляет собой изменения или добавления к предыдущему слою.

**Слой — это неизменяемый файловый снимок (snapshot) изменений, сделанных инструкцией.**

Docker использует **Union File System (OverlayFS)**, которая позволяет объединять несколько файловых систем в одну.

<br>

##### Основные аспекты, которые помогут лучше понять Docker layers:

1. **Union File System (UnionFS)**: Docker использует UnionFS, что позволяет объединять несколько файловых систем в одну. Этот подход позволяет слоям Docker быть как можно более легковесными и повторно используемыми. 
<br>

2. **Кэширование слоев:** Каждый слой кэшируется. Если в Dockerfile или процессе создания образа меняется только один шаг, все до этого шага можно использовать из кэша, что делает создание образов быстрее.
<br>

3. **Шаги в Dockerfile:** Каждый шаг в Dockerfile (например, RUN, COPY, ADD) создает новый слой. Это позволяет изолировать изменения и оптимизировать сборку.
<br>

4. **Базовый образ:** Образ обычно начинается с базового образа (например, FROM ubuntu:20.04), который сам по себе уже состоит из нескольких слоев.
<br>

5. **Полезные слои:** Каждый последующий слой хранит информацию только о различиях по сравнению с предыдущими слоями. Это позволяет Docker экономить место и упрощать процесс обновления образов.
<br>

6. **Слои только для чтения и запись:** Все слои, кроме самого последнего, являются только для чтения. Последний слой, соответствующий запущенному контейнеру, является слоем записи, куда вносятся все изменения, которые происходят, пока контейнер работает.



### FROM — задает базовый образ

dockerfile

FROM ubuntu:22.04
FROM node:18-alpine
FROM python:3.11-slim
FROM scratch          # пустой образ, для статических бинарников

### WORKDIR — устанавливает рабочую директорию

dockerfile

WORKDIR /app
# Все последующие команды (COPY, RUN, CMD) будут выполняться в /app

### COPY — копирует файлы с хоста в образ

dockerfile

COPY app.py /app/
COPY package.json package-lock.json ./
COPY --chown=node:node . /app    # с указанием владельца
COPY --from=builder /app/dist ./ # из предыдущего этапа сборки

### ADD — расширенная версия COPY

dockerfile

ADD app.tar.gz /app/     # автоматически распаковывает tar-архивы
ADD https://example.com/file.txt /tmp/  # может скачивать по URL
# Но лучше использовать COPY для локальных файлов (явнее)

### RUN — выполняет команды во время сборки

dockerfile

RUN apt-get update && apt-get install -y python3
RUN npm install
RUN pip install -r requirements.txt
RUN chmod +x entrypoint.sh

### ENV — устанавливает переменные окружения

dockerfile

ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgresql://user:pass@localhost/db

### ARG — переменные только для сборки

dockerfile

ARG VERSION=latest
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
# Передаются при сборке: docker build --build-arg VERSION=1.0.0 .

### EXPOSE — декларирует порты (не публикует, а документирует)

dockerfile

EXPOSE 3000
EXPOSE 80/tcp
EXPOSE 443/udp

### CMD — команда по умолчанию при запуске контейнера

dockerfile

CMD ["node", "app.js"]          # exec form (рекомендуется)
CMD npm start                    # shell form
CMD ["python", "app.py"]

### ENTRYPOINT — основная команда, не переопределяется при запуске

dockerfile

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["--help"]                   # аргументы по умолчанию для ENTRYPOINT

**Разница CMD и ENTRYPOINT:**

dockerfile

# ENTRYPOINT + CMD
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run my-image           → python app.py
# docker run my-image script.py → python script.py (CMD переопределяется)

### USER — меняет пользователя

dockerfile

RUN useradd -m appuser
USER appuser
WORKDIR /home/appuser

### VOLUME — создает точку монтирования для томов

dockerfile

VOLUME /data
VOLUME ["/var/log", "/var/lib/mysql"]

### LABEL — метаданные образа

dockerfile

LABEL maintainer="user@example.com"
LABEL version="1.0"
LABEL description="My awesome app"

---


## 🔧 Оптимизация и best practices

### 1. Объединяем RUN команды (меньше слоев)

dockerfile

# ❌ Плохо (3 слоя)
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean
# ✅ Хорошо (1 слой)
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

### 2. Используем .dockerignore

gitignore

# .dockerignore
node_modules/
.git/
*.log
.env
.DS_Store
__pycache__/
*.pyc

### 3. Порядок инструкций (кэширование слоев)

dockerfile

# ❌ Плохо — при изменении кода npm install будет переустанавливаться
COPY . /app
RUN npm install
# ✅ Хорошо — package.json меняется редко, зависимости кэшируются
COPY package.json package-lock.json ./
RUN npm install
COPY . ./

### 4. Multi-stage builds (многоэтапная сборка)

dockerfile

# Этап 1: сборка
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
# Этап 2: финальный образ (без инструментов сборки)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

### 5. Выбираем минимальный базовый образ

dockerfile

# Полный (большой) — 500MB+
FROM ubuntu:22.04
# Slim версия — 100-200MB
FROM python:3.11-slim
# Alpine (минимальный) — 5-50MB
FROM node:18-alpine
FROM alpine:latest
FROM golang:1.21-alpine
# Дистрибутив без glibc (для статически скомпилированных бинарников)
FROM scratch
COPY mybinary /mybinary
CMD ["/mybinary"]

### 6. Не запускаем процессы от root

dockerfile

FROM node:18-alpine
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser
CMD ["node", "app.js"]
















```shell
# Задает базовый образ
FROM python:3.10

# Выполняет команды в процесс сборки образа
RUN pip install flask

# Копируте файлы из локальной стистемы в образ
COPY app.py /app/app.py

# Задает рабочую директорию внутри контейнера
WORKDIR /app

# Определеят переменные окружения
ENV PORT=5000

# Команда при запуске контейнера
CMD ["python", "app.py"]
```



## Полезные флаги для сборки
```
# Сборка с именем и тегом
docker build -t myapp:latest .

# Сборка с указанием файла Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Передача аргументов сборки
docker build --build-arg VERSION=1.0.0 -t myapp:1.0.0 .

# Без кэша
docker build --no-cache -t myapp:latest .

# Сборка с выводом подробностей
docker build --progress=plain -t myapp:latest .
```


## Размеры образов для ориентира
```
|Базовый образ|Размер|
|---|---|
|`scratch`|0 B|
|`alpine:latest`|~5 MB|
|`busybox:latest`|~1 MB|
|`node:18-alpine`|~130 MB|
|`node:18`|~1 GB|
|`python:3.11-slim`|~130 MB|
|`python:3.11`|~1 GB|
|`ubuntu:22.04`|~77 MB|
|`debian:bookworm-slim`|~80 MB|
```

## Отладка и проверка

```
# Просмотр слоев образа
docker history myapp:latest

# Просмотр метаданных
docker inspect myapp:latest

# Запуск с переопределением команды для отладки
docker run -it myapp:latest /bin/sh

# Просмотр размера образа
docker images myapp:latest
```