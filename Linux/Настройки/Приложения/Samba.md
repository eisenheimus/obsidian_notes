##### -= На хосте proxmox =-
```bash
# Смотрим диски в систему
lsblk

# форматируем выбранный диск
mkfs.ext4 /dev/sdb1

# создаем каталог для монтирования
mkdir -p /mnt/ssd256

# монтируем диск в каталог
mount /dev/sdb1 /mnt/ssd256

# Узнаем UUID целевого диска
blkid /dev/sdb1

# сохраняем конфиг в /etc/fstab для автомонтирования после перезагруки
UUID=ваш-uuid-here /mnt/ssd256 ext4 defaults 0 2

# создаем каталог на samba сервере
# можно и на хосте samba создать каталог mkdir -p /mnt/samba_data
pct exec 100 -- mkdir -p /mnt/samba_data

# мапим каталоги 
pct set 100 -mp0 /mnt/ssd256,mp=/mnt/samba_data
# или можно сделать тоже самое в конфиге /etc/pve/lxc/100.conf 
# нужно добавить строку - mp0: /mnt/hdd1tb,mp=/mnt/samba_data

# перезапускаем контейнер
pct reboot 100
```

##### -= На хосте samba =-
```bash

useradd -m -s /bin/bash smbuser 
passwd smbuser

id smbuser -> например получим 1000

/etc/samba/smb.conf
------------------------
[SSD_Share]
   path = /mnt/samba_data
   read only = no
   writable = yes
   valid users = smbuser
   force user = root
------------------------
```



##### -= На хосте proxmox =-
```bash
chown -R 101000:101000 /mnt/ssd256 (100000 + 1000)
```


##### -= На хосте samba =-
```bash
systemctl restart smbd
```

##### -= Настройка iptables =-

```bash
# Установка
apt install iptables -y
apt install iptables-persistent -y

iptables -L -n -v

# 1. Разрешаем локальный интерфейс (обязательно первым)
iptables -A INPUT -i lo -j ACCEPT

# 2. Разрешаем SSH (чтобы не потерять управление)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 3. Разрешаем установленные соединения (ответы сервера клиентам)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 4. Разрешаем Samba TCP (файлы) только из локалки
iptables -A INPUT -p tcp --dport 445 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 139 -s 192.168.1.0/24 -j ACCEPT

# 5. Разрешаем Samba UDP (обнаружение в сети) только из локалки
iptables -A INPUT -p udp --dport 137:138 -s 192.168.1.0/24 -j ACCEPT

# включаем блокировку всего лишнего
iptables -P INPUT DROP

netfilter-persistent save

# обнулить все правила, если что-то пошло не так...
iptables -F

```