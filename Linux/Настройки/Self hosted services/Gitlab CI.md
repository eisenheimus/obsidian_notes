
```bash
# Обновляем пакеты
sudo apt update -y
```

```bash
# Устанавливаем curl для скачивания скрипта
apt install -y curl openssh-server ca-certificates tzdata perl
```

```bash
# Скачиваем и запускаем установочный скрипт Gitlab CE () с сайта gitlab
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```

```bash
# Правим  /etc/gitlab/gitlab.rb
EXTERNAL_URL="https://gitlab.example.com"

# или добавлеяем в ENV
sudo EXTERNAL_URL="https://url.com..."
```

```bash
# Чтобы не использовать временный пароль, можно испльзовать переменные окружения
# По умолчанию парль доступен /etc/gitlab/initial_root_password
sudo GITLAB_ROOT_PASSWORD="qwerty12345"
```

```bash
# Опциональные параметры
APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE="true"
```

```bash
# Установка
sudo apt instal gitlab-ce
```

```bash
# После мены парметров /etc/gitlab/gitlab.rb необходимо переконфигурировать
sudo gitlab-ctl reconfigue
```

```bash
# Загружаем главную страницу, вводим root и пароль (лежит в файле /etc/gitlab/initial_root_password) или из ENV
``` 