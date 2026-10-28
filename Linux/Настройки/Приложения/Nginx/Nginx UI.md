
```bash
sudo apt update && sudo apt upgrade -y
```

```bash
# Загрузка установочного скрипта
curl -O https://cloud.nginxui.com/install.sh
# Запуск установки
sudo bash install.sh install

# Скрипт автоматически выполнит следующие действия:
# Скачает подходящий бинарный файл для вашей архитектуры.
# Установит его в `/usr/local/bin/nginx-ui`.
# Создаст файл конфигурации по умолчанию в `/usr/local/etc/nginx-ui/app.ini`.
# Настроит и запустит службу `nginx-ui` через systemd.
# Важно:** По умолчанию интерфейс будет доступен на порту 9000, а не 8888
```


```bash
# Запустить службу Nginx UI
sudo systemctl start nginx-ui

# Остановить службу
sudo systemctl stop nginx-ui

# Перезапустить службу
sudo systemctl restart nginx-ui

# Проверить статус службы
sudo systemctl status nginx-ui
```


**Получение секретного ключа для первого входа **

```bash
# При первом запуске служба сгенерирует секретный ключ, необходимый для завершения установки через веб-интерфейс. Вы можете найти его одним из следующих способов:

sudo journalctl -u nginx-ui | grep "Secret"
# или 
sudo cat /usr/local/etc/nginx-ui/.install_secret
```


**Не оставляйте порт 9000 открытым для всего мира.** Это стандартный порт для веб-интерфейса, который не защищен HTTPS

**Рекомендуется:** Настроить ваш существующий Nginx в качестве reverse-proxy перед Nginx UI

**HTTP**
```nginx
server {
    listen 80;
    server_name nginx-ui.example.com;
    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**HTTPS**
```nginx
server {
    listen 80;
    server_name nginx-ui.ваш-домен.com;
    
    # Автоматический редирект с HTTP на HTTPS
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
    server_name nginx-ui.ваш-домен.com;
    # Пути к SSL сертификатам (замените на свои)
    ssl_certificate /etc/nginx/ssl/ваш-домен.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/ваш-домен.com/privkey.pem;
    
    # Рекомендуемые настройки SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    # Проксирование на Nginx UI (который слушает localhost:9000)
    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Важно для WebSocket (если нужно в будущем)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```


**Безопасность:**

/usr/local/etc/nginx-ui/app.ini
```
[server]
address = 127.0.0.1
port = 9000
```

```bash
# Закройте порт 9000
sudo ufw deny 9000
```


**Получите SSL сертификат** (если у вас еще нет для этого поддомена):
```bash
sudo certbot certonly --nginx -d nginx-ui.ваш-домен.com
```

**Создайте конфиг** в `/etc/nginx/sites-available/nginx-ui.conf`


```bash
# Добавьте базовую HTTP аутентификацию перед тем, как пользователь попадет в Nginx UI
sudo htpasswd -c /etc/nginx/.htpasswd admin
```

```nginx
# И добавьте в секцию `location /` внутри вашего HTTPS блока:
location / {
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    
    proxy_pass http://127.0.0.1:9000;
    # ... остальные proxy настройки
}
```