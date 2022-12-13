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
|	sda3	|	BTRFS	|	ARCHFILESYSTEM	|	VARIABLE																			|	/			|

### BTRFS Layout
```
			╔════════════════════════════════════╗
sda3			║Partition formatted as BTRFS        ║
|			╠════════════════════════════════════║
├@active		║Subvolume                           ║
|├-rootvol		║Subvolume mounted at /              ║
|└-homevol		║Subvolume mounted at /home          ║
|			╠════════════════════════════════════║
└@snapshots		║Stores Snapshots as subvolumes      ║
			╚════════════════════════════════════╝
```
