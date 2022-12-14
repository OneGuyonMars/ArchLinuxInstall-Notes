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
pacman -S fish iwd ufw polkit firefox terminus-font wget vifm fzy git wl-clipboard btop
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

### Optional Configuration
```
systemctl enable ufw iwd systemd-resolved 
```

## REBOOT
```
exit
umount -R /mnt
reboot
```

## Post Installation

### Iwd

Edit /etc/iwd/main.conf as

```
[General]
EnableNetworkConfiguration=true

[Network]
NameResolvingService=systemd
```
## Personal Configurations

### Swaywm
```
sudo pacman -S sway swaylock swayidle swaybg imv mpv foot 
```

### Intel
```
sudo pacman -S --needed mesa vulkan-intel libva-intel-driver libvdpau-va-gl libva-utils vdpauinfo intel-media-sdk
```
Edit /etc/environment
```
...
LIBVA_DRIVER_NAME=i965
VDPAU_DRIVER=va_gl
...
```

### Blacklist nvidia
edit **/etc/modprobe.d/blacklist-nouveau.conf**
```
blacklist nouveau
options nouveau modeset=0
```
edit **/etc/udev/rules.d/00-remove-nvidia.rules**
```
# Remove NVIDIA USB xHCI Host Controller devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA USB Type-C UCSI devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA Audio devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA VGA/3D controller devices
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x03[0-9]*", ATTR{power/control}="auto", ATTR{remove}="1"
```

### Thermald
```
sudo pacman -S thermald
sudo systemctl enable --now thermald
```

### Pipewire
```
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber pulsemixer
systemctl --user enable --now pipewire.socket wireplumber pipewire-pulse.socket
```

### GRUB
/etc/default/grub
```
GRUB_TIMEOUT=0
GRUB_CMDLINE_LINUX_DEFAULT="quiet loglevel=3 udev.log_level=3"
GRUB_DISABLE_OS_PROBER=false <-- Uncomment this
GRUB_TIMEOUT_STYLE=hidden
```

### Cpupower

```
sudo pacman -S cpupower
sudo systemctl enable cpupower
```
edit **/etc/default/cpupower**
```
governor='powersave'
```

### Power-profiles-daemon
```
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now  power-profiles-daemon
```
# References
[Egara](https://github.com/egara/arch-btrfs-installation)
[Mikeroyal](https://github.com/mikeroyal/PipeWire-Guide#Installing-PipeWire-on-Arch-Linux)
[LarryisBetter](https://gist.github.com/LarryIsBetter/218fda4358565c431ba0e831665af3d1)
