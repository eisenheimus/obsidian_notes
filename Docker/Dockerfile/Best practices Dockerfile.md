##### Объединяем RUN команды (меньше слоев)

```dockerfile
# ❌ Плохо (3 слоя)
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean

# ✅ Хорошо (1 слой)
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

##### Используем .dockerignore
```ini
# .dockerignore
node_modules/
.git/
*.log
.env
.DS_Store
__pycache__/
*.pyc
```

##### Порядок инструкций (кэширование слоев)
```dockerfile
# ❌ Плохо — при изменении кода npm install будет переустанавливаться
COPY . /app
RUN npm install

# ✅ Хорошо — package.json меняется редко, зависимости кэшируются
COPY package.json package-lock.json ./
RUN npm install
COPY . ./
```

##### Multi-stage builds (многоэтапная сборка)
```dockerfile
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
```

##### Выбираем минимальный базовый образ
```dockerfile
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
```

##### Не запускаем процессы от root
```dockerfile
FROM node:18-alpine
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup
WORKDIR /app
COPY --chown=appuser:appgroup . .
USER appuser
CMD ["node", "app.js"]
```

