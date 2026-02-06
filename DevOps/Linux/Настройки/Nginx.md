
*Установка nginx*
```bash
sudo apt install -y nginx
```

  

```bash

*Включаю автозапуск nginx при загрузке системы...*

sudo systemctl enable --now nginx

```

  

```bash

*Проверяю статус nginx...*

sudo systemctl status nginx

```

  

```bash

*Проверяю, что nginx в автозагрузке...*

sudo systemctl is-enabled nginx

```

  

```bash

*Удаляю старый конфиг (если существует)...*

sudo rm -f /etc/nginx/sites-enabled/trilium-proxy

```

  

```bash

*Создаю новый файл конфигурации...*

sudo nano /etc/nginx/sites-available/trilium

```

  

```nginx
# Основной блок сервера для обработки HTTP-трафика
server {
 
    # Прослушивать порт 80 (HTTP) для указанного домена
    listen 80;
    server_name koto.su;
  
    # Блокировать прямой доступ к корневому пути
    location = / {
        return 401;  # Возвращать ошибку 401 Не авторизован для доступа к корню
    }

    # Путь к приложению Trilium Notes
    location /trilium/ {
        # Перенаправлять запросы на бэкенд-сервер Trilium
        proxy_pass http://192.168.1.11:8080/;

        # Передавать оригинальный заголовок Host на бэкенд
        proxy_set_header Host $host;

        # Передавать реальный IP-адрес клиента на бэкенд
        proxy_set_header X-Real-IP $remote_addr;

        # Передавать цепочку прокси для отслеживания IP
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Сообщать бэкенду об оригинальном протоколе (HTTP)
        proxy_set_header X-Forwarded-Proto $scheme;
  
        # Использовать HTTP/1.1 для лучшей производительности
        proxy_http_version 1.1;        

        # Заголовки для WebSocket (необходимы для работы в реальном времени)
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";        

        # Длительные таймауты для WebSocket-соединений
        proxy_read_timeout 86400s;  # 24 часа
        proxy_send_timeout 86400s;  # 24 часа
    }

    # Перенаправлять /trilium на /trilium/ (добавлять слеш в конце)
    location = /trilium {
        return 301 /trilium/;  # Постоянное перенаправление на правильный путь
    }

    # Блокировать все другие пути, не определённые явно
    location / {
        return 404;  # Страница не найдена для всех остальных запросов
    }
 
}

```

  

```bash

*Создаю симлинк для активации конфига...*

sudo ln -sf /etc/nginx/sites-available/trilium /etc/nginx/sites-enabled/

```

  

```bash

*Проверяю конфигурацию nginx...*

sudo nginx -t

```

  

```bash

*Перезагружаю nginx для применения изменений...*

sudo systemctl reload nginx

```

  

```bash

*Финальная проверка статуса nginx...*

sudo systemctl status nginx

```

  

```bash

# “Проверьте логи”

sudo tail -f /var/log/nginx/error.log

```

  

```bash

  

# Проверьте доступность порта”

sudo netstat -tlnp | grep :80

```