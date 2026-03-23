
```bash
# Останавливаем контейнер
pct stop <ID>
```

``` bash
# Cоздаем каталог на хосте
sudo mkdir -p /mnt/usb_hdd
```

``` bash
# Находм подключенный диск, например sda2
lsbkl -t
```

```bash
# Монтируем подключенный диск в систему
sudo mount /dev/sda2 /mnt/usb_hdd
```

```bash
# Выдваем права на доступ www-data
sudo chown -R 100033:100033 /mnt/usb_hdd
```

```bash
# Добавляем точку монтирования/проброс для LXC
echo "mp0: /mnt/usb_hdd,mp=/media/usb" >> /etc/pve/lxc/<ID>.conf
```

```bash
# Запускаем контейнер
pct start <ID>
```

