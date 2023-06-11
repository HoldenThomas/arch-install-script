## nordvpn-bin
```
groupadd -r nordvpn
gpasswd -a <username> nordvpn
nordvpn whitelist add subnet 192.168.0.0/24
```

## pacman issues
```
pacman-key --init
pacman-key --populate
pacman-key --refresh-keys
```

### encryption
```
cryptsetup luksFormat /dev/<partition>
cryptsetup open /dev/<partition> <name>
pacstrap cryptsetup lvm2 /mnt
```
Edit /etc/mkinitcpio.conf

```
add to HOOKS encrypt lvm2
mkinitcpio -P
```

exit chroot
```
lsblk -f >> /mnt/etc/default/grub
arch-chroot /mnt
```
Edit /etc/default/grub

```
add to GRUB_CMLINE_LINUX_DEFAULT
cryptdevice=UUID=<partition uuid>:<name> root=UUID=<decrypted partition uuid>
```

## mounting drives
Edit /etc/credentials/samba/share
```
username=<user>
password=<pass>
```
```
chown root:root /etc/credentials
chmod 700 /etc/credentials/samba
chmod 600 /etc/credentials/samba/share
```
```
mount --mkdir -t cifs //<server>//<share> <mountpoint> -o credentials=/etc/credentials/samba/share,uid=<user>,gid=<group>
```
Edit /etc/fstab
```
//<server>//<share> <mountpoint> cifs _netdev,nofail,credentials=/etc/credentials/samba/share,uid=<user>,gid=<group> 0 0
```

Edit /etc/credentials/veracrypt/storage with <password>
```
chown root:root /etc/credentials
chmod 700 /etc/credentials/veracrypt
chmod 600 /etc/credentials/veracrypt/storage
```
Edit /etc/cryttab
```
<name> /dev/<partition> /etc/credentials/veracrypt/storage tcrypt,tcrypt-veracrypt,noauto
```
Edit /etc/fstab
```
/dev/mapper/<name> <mountpoint> ntfs3 noauto,x-systemd.automount,uid=<user>,gid=<group> 0 0
```
