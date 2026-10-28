
```bash
# Добавить  репозиторий#  nodesource
curl -L https://deb.nodesource.com/setup_lts.x -o setup.sh
# или конкретную версию
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
```

```bash
# Обновить пакеты
sudo apt update -y
```

```bash
# Установить nodejs
sudo apt install nodejs -y
```

```bash
# Установка через NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install --lts
```


### NVM
```bash
# Установка NVM
nvm install 23.4.0

# списком установленных в системе версий
nvm list

# переключиться с одной версии на другую
nvm use [version.number]
```


