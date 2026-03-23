```bash
# Получаем список всех usb устройств
sudo lsusb
```

```bash
# Проброс USB
qm set [VM_ID] -usb0 host=[USB_ID]

# Пример USB 2.0
qm set 100 -usb0 host=152d:2329

# Или USB 3.0
qm set 100 -usb1 host=152d:2329,usb3=1
```

```bash
# удалить usb устройство
qm set 100 -delete usb0
```