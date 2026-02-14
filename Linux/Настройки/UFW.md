
```bash
# 1. Разрешаем SSH (обязательно!)
sudo ufw allow 22/tcp

# 2. Разрешаем HTTP и HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 3. Устанавливаем политику по умолчанию: запретить всё входящее
sudo ufw default deny incoming

# 4. Устанавливаем политику по умолчанию: разрешить всё исходящее
sudo ufw default allow outgoing

# 5. Включаем UFW
sudo ufw enable

# 6. Проверяем статус
sudo ufw status verbose

# Удалить правило
sudo ufw delete allow 22/tcp
```