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
suso startx /usr/bin/mate-session

# Создайте алиас (сокращение) для быстрого запуска
echo "alias mate='startx /usr/bin/mate-session'" >> ~/.bashrc

# Обновите настройки
source ~/.bashrc

# Теперь можете запускать просто командой:
mate

```