# Arch-Installer

### Wifi
```
iwctl
	device list
	station device scan
	station device get-networks
	station device connect SSID
```
### Network Manager
```
nmtui
```
### Mounting drives
```
lsblk -f
sudo vim /etc/fstab

UUID="{efi}" /boot vfat defaults 0 2
UUID="" /run/media/{username}/{drivename} ntfs{type} defaults 0 0
//{truenas}/{share} /run/media/Truenas cifs username={user},password={password},uid={user},gid={group} 0 0
sudo mount //{truenas}/{share} /run/media/{share} -o username={user},uid={user},gid={group}
```
### Bluetooth
```
bluetoothctl
```
### Printers
```
localhost:631/admin
```
### VPN
```
import vpn
vpn settings:
	add dns servers 103.86.96.100, 103.86.99.100
connection settings:
	auto connect to vpn
	ignore ipv6
```
