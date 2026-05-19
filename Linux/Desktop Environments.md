```bash

# устанавливает загрузку в текстовый режим по умолчанию (аналог runlevel 3) 
sudo systemctl set-default multi-user.target

# немедленно останавливает графический дисплейный менеджер GDM (GNOME Display Manager)
sudo systemctl stop gdm

# Чтобы вернуть графическую загрузку
sudo systemctl set-default graphical.target 

# Какой target используется по умолчанию
systemctl get-default
```


Отключить DE
```bash
# Отключить текущий менеджер входа если GDM (Gnome)
sudo systemctl disable gdm 
 
# или если LightDM (XFCE, MATE, LXQt)
sudo systemctl disable lightdm 

# если SDDM (KDE Plasma, LXQt)
sudo systemctl disable sddm 
```


MATE
```bash
# минимальная установка (только среда)
sudo apt install -y mate-desktop-environment-core xinit

# или полная версия + софт
sudo apt install -y ubuntu-mate-desktop xinit

# Запуск оболочки
sudo startx /usr/bin/mate-session

# Создайте алиас (сокращение) для быстрого запуска
echo "alias mate='startx /usr/bin/mate-session'" >> ~/.bashrc

# Обновите настройки
source ~/.bashrc

# Теперь можете запускать просто командой:
mate
```

MATE по умолчанию 
```bash
# Проверка статуса, если inactive или dead — LightDM не запускается автоматически
sudo systemctl status lightdm

# Проверить, какие менеджеры входа установлены
dpkg -l | grep -E "gdm|lightdm|sddm"

# Отключить GDM (если он ещё включён)
sudo systemctl disable gdm
sudo systemctl stop gdm

# Включить автозапуск LightDM
sudo systemctl enable lightdm

# Запустить LightDM прямо сейчас
sudo systemctl start lightdm

# Создать/отредактировать конфиг LightDM
sudo nano /etc/lightdm/lightdm.conf
```

/etc/lightdm/lightdm.conf
```ini
[Seat:*]
user-session=mate
greeter-session=lightdm-gtk-greeter
```

```bash
# Проверить наличие файла сессии MATE
ls /usr/share/xsessions/mate.desktop

# Если файла нет — создайте его
sudo nano /usr/share/xsessions/mate.desktop
```

```ini
[Desktop Entry]
Name=MATE
Comment=This session logs you into MATE
Exec=mate-session
TryExec=mate-session
Type=Application
DesktopNames=MATE
```

```bash
# перезагрузиться
sudo reboot

# Посмотреть текущий режим загрузки
systemctl get-default

# Если выдаёт `multi-user.target` — меняем на графический:
sudo systemctl set-default graphical.target
```