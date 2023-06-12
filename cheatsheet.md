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

## youtube-dl issues
Edit /usr/lib/python3.11/site-packages/youtube_dl/extractor/youtube.py and commit out
```
'uploader_id': self._search_regex(r'/(?:channel|user)/([^/?&#]+)', owner_profile_url, 'uploader id') if owner_profile_url else None,
```

## qemu/virt manager
```
pacman -S qemu virt-manager virt-viewer dnsmasq iptables-nft vde2 bridge-utils openbsd-netcat libguestfs
systemctl enable libvirtd --now
```
Edit /etc/libvirt/libvirtd.conf and uncommit
```
unix_sock_group = "libvirt"
unix_sock_ro_perms = "0777"
unix_sock_rw_perms = "0770"
```
```
usermod -aG libvirt <user>
systemctl restart libvirtd
```
Enable XML editing in preferences
#### windows vm
Edit XML
delete these lines
```
<timer name="rtc" tickpolicy="catchup"/>
<timer name="pit" tickpolicy="delay"/>
```
change this line from no to yes
```
<timer name="hpet" present="yes"/>
```
in cpus change topology
```
Sockets: 1
Cores: 4
Threads: 2
```
change Virtual Disk 1 bus to VirtIO

change NIC Device model to VirtIO

change video to VirtIO

download drivers to recognize virtual disk (virtio-win.iso) from https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.215-2/

add the iso drivers by clicking add hardware under storage select device type cdrom and adding the path to virtio.iso

After booting install drivers for disks, ethernet, display

## ufw firewall
```
systemctl enable ufw --now
```
