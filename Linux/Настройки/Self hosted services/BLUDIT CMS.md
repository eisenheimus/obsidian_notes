https://docs.bludit.com/ru/getting-started/installation-guide

Сначала нужно установить [[Docker]]

```bash
# Скачиваем образ bludit
docker pull bludit/docker:latest
```

```bash
# Запускаем образ с вынесенным контентом в volume на 8080 порту
docker run --name bludit \
  -p 8000:80 \
  -v ~/my_bludit_site:/usr/share/nginx/html/bl-content \
  -d bludit/docker:latest
  
#или на 80 порту

docker run --name bludit \
  -p 80:80 \
  -v ~/my_bludit_site:/usr/share/nginx/html/bl-content \
  -d bludit/docker:latest
```


```ini
# Создаем service
[Unit]
Description=Bludit Docker Container
After=network.target docker.service
Requires=docker.service
[Service]
Type=simple
User=dockeruser
Group=dockeruser
Restart=always
RestartSec=5s
ExecStart=/usr/bin/docker start -a bludit
ExecStop=/usr/bin/docker stop -t 10 bludit
[Install]
WantedBy=multi-user.target
```

```bash
# применяем и перезапускаем
sudo systemctl daemon-reload
sudo systemctl enable bludit.service
sudo systemctl start bludit.service
sudo systemctl status bludit.service
```
### Важное предупреждение о правах доступа

При использовании этого подхода вы можете столкнуться с ошибкой при попытке создать новую статью: _"Writing test failure, check directory bl-content permissions"_[](https://forum.bludit.org/viewtopic.php?p=13440&sid=87c3ead2c7241c6b03e6b06ec6278c3d#p13440)[](https://github.com/bludit/docker/issues/7).

Это происходит из-за несоответствия прав доступа к файлам между пользователем внутри контейнера и вашим хост-пользователем. Проблема решается одним из двух способов:

**Способ 1: Запустить контейнер от root (простой, но менее безопасный)**  
Добавьте флаг `--user root`, чтобы контейнер работал с максимальными привилегиями: