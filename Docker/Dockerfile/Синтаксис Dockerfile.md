##### FROM — задает базовый образ
```dockerfile
FROM ubuntu:22.04
FROM node:18-alpine
FROM python:3.11-slim

# пустой образ, для статических бинарников
FROM scratch          
```


##### WORKDIR — устанавливает рабочую директорию
```dockerfile
WORKDIR /app
# Все последующие команды (COPY, RUN, CMD) будут выполняться в /app
```


##### COPY — копирует файлы с хоста в образ
```dockerfile
COPY app.py /app/
COPY package.json package-lock.json ./
COPY --chown=node:node . /app    # с указанием владельца
COPY --from=builder /app/dist ./ # из предыдущего этапа сборки
```


##### ADD — расширенная версия COPY
```dockerfile
ADD app.tar.gz /app/     # автоматически распаковывает tar-архивы
ADD https://example.com/file.txt /tmp/  # может скачивать по URL
# Но лучше использовать COPY для локальных файлов (явнее)
```


##### RUN — выполняет команды во время сборки
```dockerfile
RUN apt-get update && apt-get install -y python3
RUN npm install
RUN pip install -r requirements.txt
RUN chmod +x entrypoint.sh
```


##### ENV — устанавливает переменные окружения
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgresql://user:pass@localhost/db
```

##### ARG — переменные только для сборки
```dockerfile
ARG VERSION=latest
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
# Передаются при сборке: docker build --build-arg VERSION=1.0.0 .
```

##### EXPOSE — декларирует порты (не публикует, а документирует)
```dockerfile
EXPOSE 3000
EXPOSE 80/tcp
EXPOSE 443/udp
```

##### CMD — команда по умолчанию при запуске контейнера
```dockerfile
CMD ["node", "app.js"]          # exec form (рекомендуется)
CMD npm start                    # shell form
CMD ["python", "app.py"]
```

##### ENTRYPOINT — основная команда, не переопределяется при запуске
```dockerfile
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["--help"]                   # аргументы по умолчанию для ENTRYPOINT
```



##### Разница CMD и ENTRYPOINT
```dockerfile
# ENTRYPOINT + CMD
ENTRYPOINT ["python"]
CMD ["app.py"]

# docker run my-image           → python app.py
# docker run my-image script.py → python script.py (CMD переопределяется)
```

##### USER — меняет пользователя
```dockerfile
RUN useradd -m appuser
USER appuser
WORKDIR /home/appuser
```


##### VOLUME — создает точку монтирования для томов
```dockerfile
VOLUME /data
VOLUME ["/var/log", "/var/lib/mysql"]
```

##### LABEL — метаданные образа

```dockerfile
LABEL maintainer="user@example.com"
LABEL version="1.0"
LABEL description="My awesome app"
```