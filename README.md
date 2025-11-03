# kim-m-nguyen.github.io

## Downloading the ISO
1. Downloaded iso file via HTTP from mit.edu. Originally downloaded the file directly on my computer instead of downloading it within my VM, so I had some trouble accessing it within my VM
2. Downloaded the iso signature 
3. downloaded the b2sums.txt file into the same directory
4. verified my iso's checksums matches the official one using command:
```
b2sum -c b2sums.txt
``` 

5. Downloading the signing key from GnuPG with 
```
gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org
```

6. Verifying the signature with gpg ```
```
--verify archlinux-2025.11.01-x86_64.iso.sig archlinux-2025.11.01-x86_64.iso
```

## Setting up the VM and other pre-installation steps
7.  Setting up the VM where Arch Linux will be installed
		-Created VM on VMware Workstation
		-Set up Guest OS using ISO
		-allocated 2GB of RAM and 20GB of HDD Storage
		-struggled to successfully boot up the machine because I did not give it 2gb of RAM
8. Changed the firmware setting to UEFI
9. Verified the boot mode with command: 
```
cat /sys/firmware/efi/fw_platform_size
```
and got 64 as output
10. Set up a network connection in the live environment
	-  ensured my network interface with command: 
	- ```
		ip link
		```
	- used rfkill to make sure the card is not blocked
	- verified my connection with command:
	```
	ping -c 3 archlinux.org 
	```
11.  Updated the system clock to CDT with command:
```
timedatectl set-timezone America/Chicago
```
12.  Create partitions on virtual hard disk
```
fdisk /dev/sda
```
	- create the EFI partition
		- command: g
		- n -> enter
		- 1 -> enter
		- enter
		- +512 -> enter
		- t -> enter
		- 1
	- create the root partition
		- n -> enter
		- 2 -> enter
		- enter
		- enter
	write the changes: w
13. Format the partitions
```
mkfs.ext4 /dev/sda1
```
```
mkfs.fat -F 32 /dev/sda1
```
14. Mounting file systems
	- mount the root partition with command:
	```
	mount /dev/sda2 /mnt
	```
	- mount the EFI partition with commands:
	```
	mkdir /mnt/boot
	```
	```
	mount /dev/sda1 /mnt/boot
	```
	15. Adjusted the mirror list
		```
		nano /etc/pacman.d/mirrorlist
		```
		to display the mirror list. 
		- moved https://losangeles.mirror.pkgbuild.com/$repo/os/$arch to the top of the list
	16. Downloaded Arch Linux using command:
	```
	pacstrap -K /mnt base linux linux-firmware
	```

	17. removed the Arch Linux iso from my VM but was told that there was no bootloader found, so I had to re-mount my system (I didn't realize that step 3 of the installation guide was to be done still in my live environment, so that's why I booted up the installed system right after step 2)
		- mounted my root partition
		```
		mount /dev/sda2 /mnt
		```
		- mounted EFI partition
		```
		mkdir -p /mnt/boot
		```
		```
		mount /dev/sda1 /mnt/boot
		```
		- chroot into my system
		```
		arch-chroot /mnt
		```
		- installed the bootloader
		```
		pacman -S grub efibootmgr
		```
		```
		grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
		```
		```
		grub-mkconfig -o /boot/grub/grub.cfg
		```
		- exit chroot and reboot
	18. Set a password using
	```
	passwd
	```
	within arch-chroot /mnt
## Configurations
*I set up my root user password, but I was having network issues inside my installed system, so I had to do these configurations within arch-chroot /mnt in my ISO
19.  generated an fstab file with command:
```
genfstab -U /mnt >> /mnt/etc/fstab
```
20.  changed root into the new system
```
arch-chroot /mnt
```
21. time zone
	- set time zone to match CDT
	```
	ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
	```
	- synchronize hardware clock
	```
	hwclock --systohc
	```
22.  localization
- install nano
```
pacman -S nano
```

- edit /etc/locale.gen
```
nano /etc/locale.gen
```
- uncomment en_US.UTF-8 UTF-8
- generate the locales
```
locale-gen
```
- create /etc/locale.conf
```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```
22. network configuration
- set a host name
```
echo archvm > /etc/hostname
```
- link hostname to localhost
```
nano /etc/hosts
```
- add a line for my hostname
```
127.0.0.1 archvm.localdomain archvm
```
23. check initramfs
```
mkinitcpio -P
```
### REBOOT!!!!

## Post Installation Steps (Assignment Requirements)
24. Immediately had network issues, so I booted back into my ISO and installed NetworkManager
```
pacman -S networkmanager
```
```
systemctl enable NetworkManager
```
25. Rebooted back into the install and ran
```
systemctl start NetworkManager
```
26. ==Installing LXQt==
- install Xorg
```
pacman -S xorg xorg-xinit
```
- install LXQt and basic packages
```
pacman -S lxqt lxqt-sudo qterminal pcmanfm-qt
```
- install a display manager
```
pacman -S sddm
```
```
systemctl enable sddm
```
27. ==Create user accounts==
- ##### for myself
```
useradd -m -G wheel -s /bin/bash kimily
```
- create password for kimily (password: kimbo!)
```
passwd kimily
```
- install sudo to give sudo access
```
pacman -S sudo
```
- edit the sudoers file
```
EDITOR=nano visudo
```
- uncomment (remove #)
```
%wheel ALL=(ALL) ALL
```
- ##### for codi
```
useradd -m -G wheel -s /bin/bash codi
```
- password: codispodi
28. ==Installing fish==
```
sudo pacman -S fish
```
- enter fish shell
```
fish
```
29. Installing ssh
- download ssh
```
pacman -S openssh
```
- enable ssh service
```
systemctl enable sshd
```
```
systemctl start sshd
```
29. ==Color coding the terminal==
- run
```
fish_config
```
- python is needed to browse, so install python
```
sudo pacman -S python
```
- a web browser is needed to access fish_config, so I installed firefox
```
sudo pacman -S firefox
```
30.  ==set the system to **boot into the GUI** desktop environment==
*-this step was already done when I installed LXQt; I installed sddm and enabled it-*
31. ==Creating aliases (in fish)==
- open Fish config file
```
nano ~/.config/fish/config.fish
```
**Aliases**
##update shortcut
```
alias update='sudo pacman -Syu'

```
##use a long listing format for ls
```
alias ll='ls -la'
```
##quickly clearing terminal
```
alias c='clear'
```
- reload config so the aliases can take effect
```
source ~/.config/fish/config.fish
```
