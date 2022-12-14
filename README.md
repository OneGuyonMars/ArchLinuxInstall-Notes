#	My Notes on installing Arch Linux
##	Pre-Installation

###	The following things have been taken into consideration while making this guide.
**Firmware**
-	[ ]	BIOS
-	[x]	UEFI
-	
**Bootloader**
-	[x]	Grub
-	[ ]	Systemd-Boot
-	[ ]	EFISTUB
-	[ ]	rEFInd

**Swap**
-	[x]	Swap Partition
-	[ ]	Swapfile
-	[ ]	Zram

**File System**
-	[ ]	ext4
-	[x]	BTRFS

### File System
|	NAME	|	FSTYPE	|	LABEL(Optional)	|	SIZE																			|	MOUNTPOINTS	|
|	:--:	|	:----:	|	:------------:	|	:---------------------------------------------------------------------------:	|	-----------	|
|	sda1	|	vfat	|	EFIBOOT			|	512M																			|	/boot/efi	|
|	sda2	|	SWAP	|	SWAP			|	[GUIDE](https://help.ubuntu.com/community/SwapFaq#How_much_swap_do_I_need.3F)	|	[SWAP]		|
|	sda3	|	BTRFS	|	ARCHFS	|	VARIABLE																			|	/			|

### BTRFS Layout
```
			╔══════════════════════════════════════════╗
sda3			║Partition formatted as BTRFS              ║
|			╠══════════════════════════════════════════║
├@active		║Subvolume                                 ║
|├-root			║Subvolume mounted at /                    ║
|└-home			║Subvolume mounted at /home                ║
|			╠══════════════════════════════════════════║
├@snapshots		║Stores Snapshots as subvolumes            ║
|			╠══════════════════════════════════════════║
└@misc			║Subvolume                                 ║
 ├-cache		║Subvolume mounted at /var/cache/pacman/pkg║
 └-log			║Subvolume mounted at /var/log             ║
			╚══════════════════════════════════════════╝
```
##	Installation
Refresh the repository and update archlinux-keyring in case of an old ISO

`sudo pacman -Sy archlinux-keyring`

> :warning:	Make sure to backup your personal data before the next step if you don't want to lose it.
Partitioning the disk
Recommended methods
-	**gdisk**	[This video uses it](https://www.youtube.com/watch?v=Xynotc9BKe8)
-	**cfdisk**	[This video uses it](https://www.youtube.com/watch?v=68z11VAYMS8)

### Formatting the partitions

```
mkfs.fat -F32 -n EFIBOOT /dev/sda1
mkswap -L SWAP /dev/sda2
swapon /dev/sda2
mkfs.btrfs -L ARCHFS /dev/sda3
```

### Creating BTRFS subvolumes

```
mount /dev/sda3 /mnt
cd /mnt
btrfs subvolume create @active
btrfs subvolume create @active/root
btrfs subvolume create @active/home
btrfs subvolume create @misc
btrfs subvolume create @misc/log
btrfs subvolume create @misc/cache
btrfs subvolume create @snapshots
```

Unmount /mnt

```
cd ..
umount /mnt
```

### Mounting root and creating required directories

```
mount -o subvol=@active/root /dev/sda3 /mnt
mkdir -p /mnt/{boot/efi,home,mnt/inactive,var/log,var/cache/pacman/pkg}
```

### Mounting all the filesystems

```
mount /dev/sda1 /mnt/boot/efi
mount -o subvol=@active/home /dev/sda3 /mnt/home
mount -o subvol=@misc/cache /dev/sda3 /mnt/var/cache/pacman/pkg
mount -o subvol=@misc/log /dev/sda3 /mnt/var/log
mount -o subvol=/ /dev/sda3 /mnt/mnt/inactive
```

### Installing archlinux

```
pacstrap -K /mnt base linux linux-firmware neovim man-db intel-ucode
```

> 🗒️ neovim can be replaced by your favorite text editor
> 🗒️ intel-ucode should be replaced by the microcode of your chipset
	
### Generating Fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot into the install

```
arch-chroot /mnt
```

### Setting up timezone

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
timedatectl set-timezone Region/City (Optional)
hwclock --systohc
```

### Setting up locale

Edit **/etc/locale.gen** as

```
...
en_US.UTF-8 UTF-8
...
```

```
locale-gen
```
Edit **/etc/locale.conf** as

```
LANG=en_US.UTF-8
```

### Setting up vconsole

Edit **/etc/vconsole.conf**
```
KEYMAP=us
FONT=Some-font (Optional)
```

### Configuring Hosts

```
echo "hostname" >> /etc/hostname
```

Edit /etc/hosts as

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	hostname.localdomain	hostname
```


### Installing some miscellaneous and required packages

Important
```
pacman -S linux-headers btrfs-progs grub efibootmgr base-devel broadcom-wl-dkms
```
> 🗒️ broadcom-wl-dkms should be installed only if you have broadcom drivers and check the appropriate package [here](https://wiki.archlinux.org/title/broadcom_wireless)


Optional

```
pacman -S fish iwd ufw polkit firefox terminus-font wget vifm fzy git wl-clipboard
```

> Install wl-clipboard only if your planning to use [wayland](https://wiki.archlinux.org/title/wayland)

### Initramfs

Edit /etc/mkinitcpio.conf
Add btrfs to HOOKS
```
HOOKS=(...	btrfs)
```

Generate initramfs

```
mkinitcpio -p linux
```

### Setting up users and passwords

Create a new user by
```
useradd -mG wheel username
```

Create a password for host and users by

```
passwd
passwd username
```
Edit sudo file by
```
EDITOR=nvim visudo
```
Uncomment the line containing 
```
...
%wheel ALL=(ALL:ALL) ALL
...
```

### Installing bootloader

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## REBOOT
