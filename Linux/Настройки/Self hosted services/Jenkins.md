
```bash
# Устанавливаем JRE
sudo apt install fontconfig openjdk-21-jre
```

```bash
# качивает публичный GPG-ключ с сервера Jenkins и сохраняет его в файл
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

# Добавление репозитория
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]"  https://pkg.jenkins.io/debian-stable binary/ | sudo tee  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Обновление локальный список доступных пакетов из репозиториев
sudo apt update

# Установка Jenkins
sudo apt install jenkins
```

```bash
# Проверка статуса
sudo service jenkins status
```

```
Переходим http://192.168.1.10:8080
```

```bash
Компируем временный пароль админа и вводим в UI
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

