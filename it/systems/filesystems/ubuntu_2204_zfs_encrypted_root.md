# Ubuntu 22.04 ZFS Encrypted Root

```
sudo -s
mkdir /mnt/zfskey
cryptsetup open /dev/zvol/rpool/keystore zfskey
mount /dev/dm-0/zfskey /mnt/zfskey
```
Check zfs filesystems and choose
```
mkdir /mnt/rpool
zpool import -R /mnt/rpool rpool
cat /path/to/system.key | sudo zfs load-key -L prompt rpool
zfs mount rpool/ROOT_????
```

