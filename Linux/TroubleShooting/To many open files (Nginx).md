
```bash
# **Временное изменение (для текущей сессии)**
# Полезно для быстрой проверки, но после перезагрузки сервера настройки сбросятся.
ulimit -n 65535
```

``
```bash
# **Постоянное изменение (рекомендовано)**
# Отредактируйте файл `/etc/security/limits.conf`:
# Добавьте в конец файла следующие строки:

* soft nofile 65535
* hard nofile 65535
```

``` bash
# После увеличения системных лимитов нужно сказать Nginx использовать их. Для этого # отредактируйте главный конфигурационный файл /etc/nginx/nginx.conf
# 1. Установите директиву `worker_rlimit_nofile`
# 2: Проверьте и увеличьте `worker_connections`
```

```nginx
user nginx;
worker_processes auto;
worker_rlimit_nofile 65535;   # <-- Добавьте эту строку
events {
# `worker_connections` не может превышать `worker_rlimit_nofile`. Если они равны — это хорошая и безопасная конфигурация
    worker_connections 65535; # Должно быть <= worker_rlimit_nofile
    use epoll;               # Эффективный метод обработки соединений для Linux
    multi_accept on;         # Разрешить воркеру принимать все новые соединения за раз
}
```

```bash
# **Проверьте конфигурацию Nginx на синтаксические ошибки**
sudo nginx -t
```

```bash
# **Перезагрузите Nginx плавно (без разрыва существующих соединений):**
sudo systemctl reload nginx
# или
sudo nginx -s reload
```
