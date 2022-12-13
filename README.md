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
			╔════════════════════════════════════╗
sda3			║Partition formatted as BTRFS        ║
|			╠════════════════════════════════════║
├@active		║Subvolume                           ║
|├-root			║Subvolume mounted at /              ║
|└-home			║Subvolume mounted at /home          ║
|			╠════════════════════════════════════║
└@snapshots		║Stores Snapshots as subvolumes      ║
			╚════════════════════════════════════╝
```
##	Installation
Refresh the repository and update archlinux-keyring in case of an old ISO

`sudo pacman -Sy archlinux-keyring`

> :warning:	Make sure to backup your personal data before the next step if you don't want to lose it.
Partitioning the disk
Recommended methods
-	**gdisk**	[This video uses it](https://www.youtube.com/watch?v=Xynotc9BKe8)
-	**cfdisk**	[This video uses it](https://www.youtube.com/watch?v=68z11VAYMS8)

Formatting the partitions

```
mkfs.fat -F32 -n EFIBOOT /dev/sda1
mkswap -L SWAP /dev/sda2
swapon /dev/sda2
mkfs.btrfs -L ARCHFS /dev/sda3
```

Creating BTRFS subvolumes
```
mount /dev/sda3 /mnt
cd /mnt
btrfs subvolume create @active
btrfs subvolume create @active/root
btrfs subvolume create @active/home
btrfs subvolume create @snapshots
```

