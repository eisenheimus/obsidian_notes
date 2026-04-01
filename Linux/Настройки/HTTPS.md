
```
# Установка certbot
apt install -y certbot python3-certbot-nginx

# Получение сертификата
certbot --nginx -d planka.yourdomain.com

# Автоматическое обновление
systemctl enable certbot.timer

#После получения сертификата обновите `BASE_URL` в `.env` на `https://planka.yourdomain.com` и перезапустите сервис
```