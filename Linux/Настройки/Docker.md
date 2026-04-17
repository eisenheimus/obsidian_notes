
##### Установка Docker
```bash
#  Обновляем индексы пакетов apt
sudo apt update
```

```bash
# Устанавливаем дополнительные пакеты
sudo apt install curl software-properties-common ca-certificates apt-transport-https -y
```

``` bash
# Импортируем GPG-ключ
wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | sudo tee /etc/apt/keyrings/docker.gpg > /dev/null
```

```bash
 # Добавляем репозиторий докера
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable"| sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
# В очередной раз обновляем индексы пакетов
sudo apt update
```

```bash
# Проверяем репозиторий
apt-cache policy docker-ce
```

```bash
# Устанавливаем докер
sudo apt install docker-ce -y
```

```bash
# Проверив статус Docker в системе
sudo systemctl status docker
```

```bash
# Создаем пользвателя 
sudo useradd -m -s /bin/bash dockeruser

# Добавляем в группу docker - докером могу правлять или группа docker или sudo
sudo usermod -aG docker dockeruser

# назначаем пароль для dockeruser
sudo passwd dockeruser

# проверям доступность командо
docker ps
```


##### Установка Docker Compose
```bash
# Установка с помощью apt-get (не последняя версия)
sudo apt-get install docker-compose

# Ручная установка послденей версии
sudo curl -L "https://github.com/docker/compose/releases/download/v2.38.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Выдаем права да доступ
sudo chmod +x /usr/local/bin/docker-compose
```