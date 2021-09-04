# Arch-Installer

### Wifi
```
iwctl
	device list
	station device scan
	station device get-networks
	station device connect SSID

nmtui
```
### Mounting drives
```
lsblk -f
sudo vim /etc/fstab
UUID="" /run/media/{username}/{drivename} ntfs{type} defaults 0 0
//{truenas}/{share} /run/media/Truenas cifs username={user},password={password},uid={user},gid={group} 0 0
```
### Printers
```
localhost:631/admin
```
