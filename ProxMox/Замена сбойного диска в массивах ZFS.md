```bash
# Смотрим список хранилищ (пулов) ZFS:
zpool list
```

``` bash
# Смотрим информацию о пуле

# UNAVAIL - умерший диск
# DEGRADED - сбойный диск
zpool status [pull_name]
```

``` bash
# Узнаем id нового диска и сбойного
ls -al /dev/disk/by-id
```

```bash
# Выполнить замену диска в пулле
zpool replace vm-store-ssd /dev/disk/by-id/ata-ADATA_SU750_2M***UA  /dev/disk/by-id/ata-Netac_SSD_512GB_AA***68

# Нужно дождаться окончания синхронизации ZFS
```

```
# Проверяем статуса пулла
zpool status vm-store-ssd
```


```bash
# сбросить ошибки массива
zpool clear vm-store-ssd
```