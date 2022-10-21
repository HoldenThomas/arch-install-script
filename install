#!/bin/sh

baseArch() {
  echo "Enter hostname" && read hostname
  echo "Enter username" && read user
  echo "Enter password" && read password
	devicelist=$(lsblk -dplnx size -o name,size | grep -Ev "boot|rpmb|loop" | tac)
  echo -e "${devicelist}\nEnter install disk(/dev/sd?" && read device
  echo "Grub removable(y,n)" && read grubRemovable
  echo -e "Video drivers\n1.\tVirtualbox\n2.\tNvidia\n3.\tAMD\n4.\tIntel\n" && read videoDriver
  echo -e "CPU microcode\n1.\tIntel\n2.\tAMD" && read cpu

	timedatectl set-ntp true

	echo -e "g\n  n\n\n\n+260M\nt\n1\n  n\n\n\n+2G\nt\n\n19\n  n\n\n\n\n  w\n" | fdisk $device
	partBoot="${device}1"
	partSwap="${device}2"
	partLinux="${device}3"
	mkfs.fat -F 32 $partBoot
	mkfs.ext4 $partLinux
	mkswap $partSwap
	swapon $partSwap
	mount $partLinux /mnt
	mkdir /mnt/boot
	mount $partBoot /mnt/boot

	pacstrap /mnt base base-devel linux linux-firmware man-db man-pages texinfo networkmanager neovim git grub efibootmgr
  [ $videoDriver -eq 1 ] && pacstrap /mnt virtualbox-guest-utils
  [ $videoDriver -eq 2 ] && pacstrap /mnt nvidia nvidia-settings
  [ $videoDriver -eq 3 ] && pacstrap /mnt xf86-video-amdgpu
  [ $videoDriver -eq 4 ] && pacstrap /mnt xf86-video-intel
	[ $cpu -eq 1 ] && pacstrap /mnt intel-ucode
  [ $cpu -eq 2 ] &&  pacstrap /mnt amd-ucode

	genfstab -U /mnt >> /mnt/etc/fstab
	arch-chroot /mnt ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime
	arch-chroot /mnt hwclock --systohc
	sed -i "177 s/^##*//" /mnt/etc/locale.gen
	arch-chroot /mnt locale-gen
	echo LANG=en_US.UTF-8 >> /mnt/etc/locale.conf
	echo $hostname >> /mnt/etc/hostname
	arch-chroot /mnt systemctl enable NetworkManager

	[ $grubRemovable = "y" ] && arch-chroot /mnt grub-install --target=x86_64-efi --efi-directory=boot --removable || arch-chroot /mnt grub-install --target=x86_64-efi --efi-directory=boot --bootloader-id=GRUB
	sed -i "s/GRUB_TIMEOUT=5/GRUB_TIMEOUT=1/" /mnt/etc/default/grub
	arch-chroot /mnt grub-mkconfig -o /boot/grub/grub.cfg

	[ $virtualBox -eq 0 ] && arch-chroot /mnt systemctl enable vboxservice

	arch-chroot /mnt useradd -m -G wheel,audio,disk,input,kvm,optical,scanner,storage,video $user
	sed -i "82 s/^##*//;s|# %wheel ALL=(ALL) NOPASSWD: ALL|%wheel ALL=(ALL) NOPASSWD: /usr/bin/mount,/usr/bin/umount,/usr/bin/pacman,/usr/bin/make,/usr/bin/nvim|" /mnt/etc/sudoers
	echo -e "${password}\n${password}" | arch-chroot /mnt passwd
	echo -e "${password}\n${password}" | arch-chroot /mnt passwd $user

	sed -i "33 s/^##*//;93,94 s/^##*//;37i ILoveCandy" /mnt/etc/pacman.conf
	arch-chroot /mnt pacman -Sy

  cp install /mnt/home/"$user"

  echo "-----------Base Install Finished-----------"
}

customize() {
  echo "Dualbooting windows(y,n)" && read timeFix
	[ $timeFix = "y" ] && sudo timedatectl set-local-rtc true --adjust-system-clock

	git clone https://aur.archlinux.org/yay.git; cd yay; makepkg --noconfirm -si; cd; rm -rf yay

	mkdir .dotfiles
	git clone --bare git://git.holdenthomas.xyz/dotfiles ~/.dotfiles
	git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME checkout
	git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME config status.showUntrackedFiles no

	options=()
	options+=("xorg-server" "graphical server" on)
	options+=("xorg-xinit" "starting graphical server" on)
	options+=("xwallpaper" "setting wallpaper" on)
	options+=("xclip" "c&p from cl" on)
	options+=("xcape" "remapping escape/super" on)
	options+=("arandr" "change monitor arragement" on)
	options+=("brightnessctl" "control screen brightness" on)
	options+=("pulseaudio-alsa" "audio system" on)
	options+=("pulsemixer" "audio controller" on)
	options+=("pamixer" "cl audio interface" on)
	options+=("alsa-utils" "audio system utils" on)
	options+=("pulseaudio-bluetooth" "bluetooth" on)
	options+=("bluez" "bluetooth" on)
	options+=("bluez-utils" "bluetooth" on)
	options+=("noto-fonts-cjk" "asian fonts" on)
	options+=("ttf-font-awesome" "emojis" on)
	options+=("ttf-joypixels" "emojis" on)
	options+=("dunst" "notification system" on)
	options+=("libnotify" "desktop notifications" on)
	options+=("picom" "transparency" on)
	options+=("reflector" "updating pacman mirrors" on)
	options+=("ntfs-3g" "accessing ntfs-partitions" on)
	options+=("cifs-utils" "accessing smba shares" on)
	options+=("p7zip" "extracting & archiving" on)
	options+=("openssh" "ssh" on)
	options+=("rsync" "syncing" on)
	options+=("zsh" "shell" on)
	options+=("starship" "shell prompt" on)
	options+=("dash" "faster /bin/sh" on)
	options+=("fzf" "fuzzy finder" on)
	options+=("bc" "cl calulator" on)
	options+=("htop" "system monitor" on)
	options+=("bashtop" "system monitor" on)
	options+=("python-psutil" "required for bashtop" on)
	options+=("neofetch" "display system info" on)
	options+=("mpv" "media player" on)
	options+=("sxiv" "image viewer" on)
	options+=("pcmanfm-gtk3" "gui file manager" on)
	options+=("gvfs" "allows pcmanfm to show other drives" on)
	options+=("arc-gtk-theme" "GTK theme" on)
	options+=("breeze-icons" "GTK icons theme" on)
	options+=("maim" "screenshots" on)
	options+=("ueberzug" "lf image previews" on)
	options+=("mediainfo" "lf media info preview" on)
	options+=("bat" "lf text preview" on)
	options+=("lynx" "lf html preview" on)
	options+=("atool" "lf zip preview" on)
	options+=("youtube-dl" "downloading youtube videos" on)
	options+=("keepassxc" "password manager" on)
	options+=("nextcloud-client" "nextcloud client" on)
	options+=("gnome-keyring" "for storing application passwords" on)
	options+=("seahorse" "for editing gnome-keyrings" on)
	options+=("imwheel" "changing mouse scroll speed" on)
	options+=("zathura" "document viewer" on)
	options+=("zathura-pdf-mupdf" "allows zathura to view epub,pdf,xps" on)
	options+=("npm" "Nodejs packaged modules for nvim coc" on)
	options+=("python-pynvim" "Python neovim module for nvim coc" on)
	options+=("ccls" "C language server for nvim coc" on)
	options+=("redshift" "changes color temp according to time" on)
	options+=("tmux" "terminal multiplexer" on)
	options+=("figlet" "create bubble letters in terminal" on)
	options+=("sxhkd" "hot key deamon" on)
	options+=("zathura-cb" "allows zathura to view comics" off)
	options+=("thunderbird" "email client" off)
	options+=("liferea" "rss reader" off)
	options+=("qbittorrent" "torrent client" off)
	options+=("networkmanager-openvpn" "configuring openvpn" off)
	options+=("network-manager-applet" "configuring openvpn" off)
	options+=("code" "microsoft code" off)
	options+=("steam" "videogames client" off)
	options+=("discord" "discord client" off)
	options+=("liquidctl" "kraken cpu cooler control" off)
	options+=("mcomix" "manga reader" off)
	options+=("cups" "printers" off)
	options+=("ghostscript" "required for my printer" off)
	sel=$(whiptail --backtitle "$apptitle" --title "Pacman Applications :" --checklist "Choose what you want" --cancel-button "Back" 0 0 0 \
	  "${options[@]}" \
	  3>&1 1>&2 2>&3)
	if [ ! "$?" = "0" ]; then
	  exit 1
	fi
	for itm in $sel; do
	  case $itm in
	    *) pkg="$pkg $(echo $itm | sed 's/"//g')";;
	  esac
	done

	optionsA=()
	optionsA+=("zsh-fast-syntax-highlighting" "shell syntax highlighting" on)
	optionsA+=("dashbinsh" "adds dash to /bin/sh" on)
	optionsA+=("gotop" "system monitor" on)
	optionsA+=("lf" "cl file manager" on)
	optionsA+=("brave-bin" "internet browser" on)
	optionsA+=("vidir" "cl bulkrename utility" on)
	optionsA+=("devour" "cl application swallowing" on)
	optionsA+=("android-file-transfer" "accessing android devices" on)
	optionsA+=("hakuneko-desktop" "manga downloader" off)
	optionsA+=("freefilesync-bin" "syncing client" off)
	optionsA+=("epson-inkjet-printer-escpr" "my printer driver" off)
	selA=$(whiptail --backtitle "$apptitle" --title "AUR Applications :" --checklist "Choose what you want" --cancel-button "Back" 0 0 0 \
	  "${optionsA[@]}" \
	  3>&1 1>&2 2>&3)
	if [ ! "$?" = "0" ]; then
	  exit 1
	fi
	for itm in $selA; do
	  case $itm in
	    *) pkgA="$pkgA $(echo $itm | sed 's/"//g')";;
	  esac
	done

	optionsG=()
	optionsG+=("dwmblocks" "dmw status bar" on)
	optionsG+=("dmenu" "my build of dmenu" on)
	optionsG+=("st" "my build of st" on)
	optionsG+=("dwm" "my build of dwm" on)
	optionsG+=("slock" "my build of slock" on)
	selG=$(whiptail --backtitle "$apptitle" --title "Git Applications :" --checklist "Choose what you want" --cancel-button "Back" 0 0 0 \
	  "${optionsG[@]}" \
	  3>&1 1>&2 2>&3)
	if [ ! "$?" = "0" ]; then
	  exit 1
	fi
	for itm in $selG; do
	  case $itm in
	    *) pkgG="$pkgG $(echo $itm | sed 's/"//g')";;
	  esac
	done

	sudo pacman --noconfirm --needed -S $pkg
	yay -S --noconfirm $pkgA
	for item in $pkgG; do
		git clone git://git.holdenthomas.xyz/$item ~/.local/src/$item && cd ~/.local/src/$item && sudo make clean install && cd
	done

	for itm in $sel; do
	  case $itm in
	    '"zsh"') chsh -s /usr/bin/zsh && mkdir -p ~/.cache/zsh;;
	    '"bluez"') sudo systemctl enable bluetooth;;
	    '"cups"') sudo systemctl enable cups;;
	  esac
	done

echo "-----------Costomization Finished - Eject Install Media and Reboot-----------"
}

echo "-----------Welcome to my Arch installscript-----------"
echo -e "1.\tBase install\n2.\tCustomize" && read i
[ $i -eq 1 ] && baseArch
[ $i -eq 2 ] && customize
