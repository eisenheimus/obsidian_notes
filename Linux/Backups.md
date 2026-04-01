Создайте скрипт для бэкапа:
```
#!/bin/bash
# /root/backup-planka.sh

BACKUP_DIR="/backup/planka"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Бэкап базы данных
sudo -u postgres pg_dump planka > $BACKUP_DIR/planka_db_$DATE.sql

# Бэкап файлов (если есть загруженные изображения)
tar -czf $BACKUP_DIR/planka_uploads_$DATE.tar.gz /var/www/planka/server/public/uploads 2>/dev/null

# Удаление старых бэкапов (старше 30 дней)
find $BACKUP_DIR -name "*.sql" -mtime +30 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
```

Сделайте скрипт исполняемым и добавьте в cron:
```
chmod +x /root/backup-planka.sh
echo "0 2 * * * /root/backup-planka.sh" | crontab -
```