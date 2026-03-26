### Статический IP
```ini
# /etc/network/interfaces

# Пример конфигурации для статического IP

# автоматически поднимать интерфейс `lo` (loopback) при загрузке
auto lo 

# настройка loopback-интерфейса (localhost, 127.0.0.1)
iface lo inet loopback

# автоматически поднимать интерфейс `ens18` при загрузке
auto ens18

# это имя физического сетевого интерфейса 
# ens - Ethernet Network Stack
# 18 - номер интерфейса (может быть `ens33`, `enp2s0`, `eth0` и т.д.)
iface ens18 inet manual

# автоматически поднимать мост `vmbr0` при загрузке
auto vmbr0

# vmbr0 - это виртуальный сетевой мост (Proxmox Virtual Bridge 0)
# inet static - статическая конфигурация IP (не DHCP)
iface vmbr0 inet static
    address 192.0.2.10/24
    gateway 192.0.2.1
    
    # bridge-ports — указывает, какие физические интерфейсы входят в мост
    # Здесь ens18 — физическая сетевая карта, которая "привязана" к мосту
    bridge-ports ens18
    
    # bridge-stp — Spanning Tree Protocol, off — отключено (рекомендуется для простых конфигураций)
    bridge-stp off
    
    # Forwarding Delay (задержка пересылки). 0 — нулевая задержка (пакеты начинают пересылаться сразу)
    bridge-fd 0
```


### Динамический IP (DHCP)
```ini
# /etc/network/interfaces

auto ens18
iface ens18 inet manual

auto vmbr0
iface vmbr0 inet dhcp
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0
```


### Если несколько сетевых карт:
```ini
# /etc/network/interfaces

# WAN интерфейс
auto ens18
iface ens18 inet manual

# LAN интерфейс
auto ens19
iface ens19 inet manual

# Мост для WAN
auto vmbr0
iface vmbr0 inet static
    address 192.168.1.10/24
    gateway 192.168.1.1
    bridge-ports ens18
    bridge-stp off
    bridge-fd 0

# Мост для LAN (VM будут во внутренней сети)
auto vmbr1
iface vmbr1 inet static
    address 10.0.0.1/24
    bridge-ports ens19
    bridge-stp off
    bridge-fd 0
```

### Проверка конфигурации
```
# Перезапустить сеть
systemctl restart networking

# Проверить состояние интерфейсов
ip a show vmbr0
ip a show ens18

# Проверить, что мост работает
brctl show

# Проверить маршрутизацию
ip route show
```


### Как это работает в Proxmox
```
Физический сервер
    │
    ├── ens18 (физическая сетевая карта)
    │       │
    │       └── vmbr0 (виртуальный коммутатор Proxmox)
    │               │
    │               ├── IP хоста: 192.0.2.10/24
    │               │
    │               ├── VM1 (виртуальная машина)
    │               │     └── виртуальный сетевой интерфейс
    │               │
    │               ├── VM2 (виртуальная машина)
    │               │     └── виртуальный сетевой интерфейс
    │               │
    │               └── другие VM...
    │
    └── выход в физическую сеть через ens18
```

### Типичные имена интерфейсов
```
ens33, ens192 — современные имена (systemd)
eth0, eth1 — старые имена
enp2s0 — PCI-based имена
```