# Installation guide
### 1. Pre-installation

#### 1.1 Acquire an installation image
Visit the  [Download](https://archlinux.org/download/)  page and, depending on how you want to boot, acquire the ISO file or a netboot image, and the respective  [GnuPG](https://wiki.archlinux.org/title/GnuPG "GnuPG")  signature.

#### 1.2 Prepare an installation medium
The ISO can be supplied to the target machine via a  [USB flash drive](https://wiki.archlinux.org/title/USB_flash_installation_medium "USB flash installation medium"), an  [optical disc](https://wiki.archlinux.org/title/Optical_disc_drive#Burning "Optical disc drive")  or a network with  [PXE](https://wiki.archlinux.org/title/PXE "PXE"): follow the appropriate article to prepare yourself an installation medium from the ISO file.

#### 1.3 Set the consolefont
[Console fonts](https://wiki.archlinux.org/title/Console_fonts "Console fonts")  are located in  `/usr/share/kbd/consolefonts/`  and can likewise be set with  [setfont(8)](https://man.archlinux.org/man/setfont.8)  omitting the path and file extension. For example, to use one of the largest fonts suitable for  [HiDPI screens](https://wiki.archlinux.org/title/HiDPI#Linux_console_(tty) "HiDPI"), run:
```
$ setfont ter-132b
```

#### 1.4 Connect to the internet
To set up a network connection in the live environment, go through the following steps:
To get an interactive prompt do:
```
$ iwctl
```
First, if you do not know your wireless device name, list all Wi-Fi devices:
```
[iwd]# device list
```
Then, to initiate a scan for networks (note that this command will not output anything):
```
[iwd]# station name scan
```
You can then list all available networks:
```
[iwd]# station name get-networks
```
Finally, to connect to a network:
```
[iwd]# station name connect SSID
```
#### 1.5 Partition the disks
When recognized by the live system, disks are assigned to a [block device](https://wiki.archlinux.org/title/Block_device "Block device") such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`. To identify these devices, use [lsblk](https://wiki.archlinux.org/title/Lsblk "Lsblk").
```
$ lsblk
```
The following [partitions](https://wiki.archlinux.org/title/Partition "Partition") are **required** for a chosen device:
__For UEFI with GPT__:
-   Root partition (`/`): Remainder of device (at least 23–32 GiB)
-   EFI system partition: 1 GiB
-   Swap partition: At least 4 GiB

__For BIOS with MBR__:
-   Root partition (`/`): Remainder of device (at least 23–32 GiB)
-   Swap partition: At least 4 GiB

Use a [partitioning tool](https://wiki.archlinux.org/title/Partitioning#Partitioning_tools "Partitioning") like `cfdisk` to modify partition tables.
```
$ cfdisk
```
#### 1.6 Format the partitions
```
# Format root partition (e.g., Ext4)
$ mkfs.ext4 /dev/root_partition

# Initialize swap partition
$ mkswap /dev/swap_partition

# Format EFI system partition to FAT32
$ mkfs.fat -F 32 /dev/efi_system_partition
```

#### 1.7 Mount the file systems
[Mount](https://wiki.archlinux.org/title/Mount "Mount")  the root volume to  `/mnt`. For example, if the root volume is  `/dev/root_partition`:
```
$ mount /dev/root_partition /mnt
```
For UEFI systems, mount EFI system partition:
```
$ mount --mkdir /dev/efi_system_partition /mnt/boot
```
If you created a [swap](https://wiki.archlinux.org/title/Swap "Swap") volume, enable it with [swapon(8)](https://man.archlinux.org/man/swapon.8):
```
$ swapon /dev/swap_partition
```

### 2. Installation
Install essential packages, Use the [pacstrap(8)](https://man.archlinux.org/man/pacstrap.8) script to install the [base](https://archlinux.org/packages/?name=base) package, Linux [kernel](https://wiki.archlinux.org/title/Kernel "Kernel") and firmware for common hardware:
```
# For BIOS
$ pacstrap -K /mnt base linux linux-firmware base-devel grub nano networkmanager

# For UEFI (add efibootmgr)
$ pacstrap -K /mnt base linux linux-firmware base-devel grub nano efibootmgr networkmanager
```
---
### 3. Configure the system

#### 3.1 Fstab
Generate an [fstab](https://wiki.archlinux.org/title/Fstab "Fstab") file (use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/title/UUID "UUID") or labels, respectively):
```
$ genfstab -U /mnt >> /mnt/etc/fstab
``` 

#### 3.2 Chroot
Time [Change root](https://wiki.archlinux.org/title/Change_root "Change root") into the new system:
```
$ arch-chroot /mnt
``` 
#### 3.3 Time
Set the [time zone](https://wiki.archlinux.org/title/Time_zone "Time zone") and synchronize hardware clock [hwclock(8)](https://man.archlinux.org/man/hwclock.8):
```
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc
```

#### 3.4 Localization
[Edit](https://wiki.archlinux.org/title/Textedit "Textedit")  `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 [locales](https://wiki.archlinux.org/title/Locale "Locale"). Generate & edit the locales by running:
```
# Edit locale.gen file
$ nano /etc/locale.gen

# Generate locales
$ locale-gen

# Create locale.conf and set LANG variable
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Set console keyboard layout
$ echo "KEYMAP=us" > /etc/vconsole.conf
```

#### 3.5 Network configuration
[Create](https://wiki.archlinux.org/title/Create "Create")  the  [hostname](https://wiki.archlinux.org/title/Hostname "Hostname")  file:
```
$ echo "yourhostname" > /etc/hostname
```
Complete the [network configuration](https://wiki.archlinux.org/title/Network_configuration "Network configuration") for the newly installed environment. That may include installing suitable [network management](https://wiki.archlinux.org/title/Network_management "Network management") software, configuring it if necessary and enabling its systemd unit so that it starts at boot.

#### 3.6 Root password
Set the root  [password](https://wiki.archlinux.org/title/Password
"Password"):
```
$ passwd
``` 
#### 3.7 Add user (Optional)
To add a new user, use the _useradd_ command:
```
# Create the new user
$ useradd -m -G wheel -s /bin/bash name

# Set password for the new user
$ passwd name
``` 
#### 3.8 Setup sudo
Configure sudo privileges:
```
$ EDITOR=nano visudo
# Uncomment "%wheel ALL=(ALL) ALL" to allow sudo access
```

#### 3.9 Setup NetworkManager
Enable [NetworkManager](https://wiki.archlinux.org/title/NetworkManager "NetworkManager") service:
```
$ systemctl enable NetworkManager
```

### 4. Boot loader
Install and configure GRUB bootloader:
```
$ grub-install /dev/sda
$ grub-mkconfig -o /boot/grub/grub.cfg
```

### 5. Reboot
Exit the chroot environment by typing  `exit`  or pressing  `Ctrl+d`.

Optionally manually unmount all the partitions with  `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with  [fuser(1)](https://man.archlinux.org/man/fuser.1).

Finally, restart the machine by typing  `reboot`: any partitions still mounted will be automatically unmounted by  _systemd_. Remember to remove the installation medium and then login into the new system with the root account.

After rebooting, remove the installation medium and login to the new system with the root account.
