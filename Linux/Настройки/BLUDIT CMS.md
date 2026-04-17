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

### Важное предупреждение о правах доступа

При использовании этого подхода вы можете столкнуться с ошибкой при попытке создать новую статью: _"Writing test failure, check directory bl-content permissions"_[](https://forum.bludit.org/viewtopic.php?p=13440&sid=87c3ead2c7241c6b03e6b06ec6278c3d#p13440)[](https://github.com/bludit/docker/issues/7).

Это происходит из-за несоответствия прав доступа к файлам между пользователем внутри контейнера и вашим хост-пользователем. Проблема решается одним из двух способов:

**Способ 1: Запустить контейнер от root (простой, но менее безопасный)**  
Добавьте флаг `--user root`, чтобы контейнер работал с максимальными привилегиями: