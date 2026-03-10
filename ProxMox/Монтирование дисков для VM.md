Фабула
1. Проверить наличие диска - lsblk -f
2. Остановить VM
3. Размонтировать диск на хосте proxmox (если подключен) - umount /dev/sdb1
4. Создать раздел - fdisk /dev/sdb
5. Внести раздел в конфиг -/etc/pve/qemu-server/[VMID].conf

```bash
# Останавливаем VM
qm stop [VMID]
```


```bash
# Проверям наличие диска
lsblk
```


```bash
# Размонтируем диск на хосте Proxmox (если он смонтирован)
umount /mnt/hdd_logs
```


```bash
# На хосте Proxmox отредактируйте конфиг VM
nano /etc/pve/qemu-server/[VMID].conf

# нужно добавить строку
scsi1: /dev/sdb
```


```bash
# Проверка
df -h /mnt/logs
```


```shell
# Пример конфига
boot: order=scsi0;ide2;net0
cores: 2
cpu: x86-64-v2-AES
ide2: local:iso/ubuntu-24.04.3-live-server-amd64.iso,media=cdrom,size=3226020K
memory: 4096
meta: creation-qemu=10.1.2,ctime=1773116529
name: monitoring-192.168.1.13
net0: virtio=BC:24:11:BA:82:CB,bridge=vmbr0,firewall=1
numa: 0
onboot: 1
ostype: l26
scsi0: local-lvm:vm-103-disk-0,iothread=1,size=30G
scsi1: /dev/sdb
scsihw: virtio-scsi-single
smbios1: uuid=ad15eba7-3289-42f8-9c61-b44bbcae5e91
sockets: 1
vmgenid: 4cbf6361-8736-4338-8d21-fd347e25ee7f
```