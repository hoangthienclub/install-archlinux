## Arch Installation Guide

ℹ️ For help with the installation process watch my video: https://www.youtube.com/watch?v=cOobSmI-XgA&t=399s

### Console keyboard layout

Find out which keyboard layout you are using and then set it using `loadkeys`:

```
$ ls /usr/share/kbd/keymaps/**/*.map.gz
$ loadkeys de_CH-latin1
```

### Console font

This step is not really necessary, but the Terminus font may appear cleaner than the default one:

```
$ setfont Lat2-Terminus16
```

### Partitioning

Check the name of the hard disk:

```
fdisk -l
```

Use the name (in my case _vda_) to start the `fdisk` partitioning tool:

```
fdisk /dev/vda
```

#### UEFI or BIOS?

Run the following command:

```
$ ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. Else you have to use BIOS mode.

#### UEFI with GPT

Press <kbd>g</kbd> to create a new GPT Partition Table.

We will do it according to the example layout of the Arch wiki:

| Mount point | Partition                   | Partition type | Suggested size      |
| ----------- | --------------------------- | -------------- | ------------------- |
| /mnt/boot   | /dev/_efi_system_partition_ | uefi           | At least 300 MiB    |
| [SWAP]      | /dev/_swap_partition_       | swap           | More than 512 MiB   |
| /mnt        | /dev/_root_partition_       | linux          | Remainder of device |

##### Create boot partition

1. Press <kbd>n</kbd>.
2. Press <kbd>Enter</kbd> to use the default first sector.
3. Enter _+300M_ for the last sector.
4. Press <kbd>t</kbd> and choose 1 and write _uefi_.

##### Create swap partition

1. Press <kbd>n</kbd>.
2. Press <kbd>Enter</kbd> to use the default first sector.
3. Enter _+512M_ for the last sector.
4. Press <kbd>t</kbd> and choose 2 and write _swap_.

##### Create root partition

1. Press <kbd>n</kbd>.
2. Press <kbd>Enter</kbd> to use the default first sector.
3. Enter <kbd>Enter</kbd> to use the default last sector.
4. Press <kbd>t</kbd> and choose 3 and write _linux_.

⚠️\* **When you are done partitioning don't forget to press <kbd>w</kbd> to save the changes!**

After partitioning check if the partitions have been created using `fdisk -l`.

##### Partition formatting

```
$ mkfs.ext4 /dev/root_partition
$ mkswap /dev/swap_partition
$ mkfs.fat -F 32 /dev/efi_system_partition
```

##### Mounting the file system

```
$ mount /dev/root_partition /mnt
$ mount --mkdir /dev/efi_system_partition /mnt/boot
$ swapon /dev/swap_partition
```

#### BIOS with MBR

Press <kbd>o</kbd> to create a new GPT Partition Table.

We will do it according to the example layout of the Arch wiki:

| Mount point | Partition             | Partition type | Suggested size      |
| ----------- | --------------------- | -------------- | ------------------- |
| [SWAP]      | /dev/_swap_partition_ | swap           | More than 512 MiB   |
| /mnt        | /dev/_root_partition_ | linux          | Remainder of device |

##### Create swap partition

1. Press <kbd>n</kbd>.
2. Press <kbd>Enter</kbd> to select the default primary partition type.
3. Press <kbd>Enter</kbd> to use the default first sector.
4. Enter _+512M_ for the last sector.
5. Press <kbd>t</kbd> and choose 1 and write _swap_.

##### Create root partition

1. Press <kbd>n</kbd>.
2. Press <kbd>Enter</kbd> to select the default primary partition type.
3. Press <kbd>Enter</kbd> to use the default first sector.
4. Enter <kbd>Enter</kbd> to use the default last sector.
5. Press <kbd>t</kbd> and choose 2 and write _linux_.

##### Make partition bootable

Press <kbd>a</kbd> and choose 2 to make the root partition bootable.

⚠️\* **When you are done partitioning don't forget to press <kbd>w</kbd> to save the changes!**

After partitioning check if the partitions have been created using `fdisk -l`.

##### Partition formatting

```
$ mkfs.ext4 /dev/root_partition
$ mkswap /dev/swap_partition
```

##### Mounting the file system

```
$ mount /dev/root_partition /mnt
$ swapon /dev/swap_partition
```

### Package install

For a minimal system download and install these packages:

```
$ pacstrap -K /mnt base base-devel linux linux-firmware e2fsprogs dhcpcd networkmanager sof-firmware git neovim man-db man-pages texinfo
```

### Last steps

#### Generate fstab file

```
$ genfstab -U /mnt >> /mnt/etc/fstab
```

#### Change root into new system

```
$ arch-chroot /mnt
```

#### Set time zone

```
$ ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
$ hwclock --systohc
```

#### Localization

Edit _/etc/locale.gen_ and uncomment _en_US.UTF-8 UTF-8_ and other needed locales. Generate the locales by running:

```
$ locale-gen
```

Create _/etc/locale.conf_ and set the _LANG_ variable according to your preferred language:

```
LANG=de_CH.UTF-8
```

Create _/etc/vconsole.conf_ and set the following variables according to your preferred language:

```
KEYMAP=de_CH-latin1
FONT=Lat2-Terminus16
```

#### Network configurations

Create _/etc/hostname_ and type any name you wish as your hostname:

```
arch
```

Edit _/etc/hosts_ like this:

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch (your host name here!)
```

#### Initramfs

```
$ mkinitcpio -P
```

#### Root password

Set a new password for root:

```
$ passwd
```

#### Bootloader

##### UEFI

Install `grub` and `efibootmgr`:

```
$ pacman -S grub efibootmgr
```

Run the following command:

```
$ grub-install --efi-directory=/boot --bootloader-id=GRUB
```

Then create a **GRUB** config file:

```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

##### BIOS

Install `grub`:

```
$ pacman -S grub
```

Check using `fdisk -l` to see the name of the disk (**not partition!**) and run the following command:

```
$ grub-install /dev/sdX
```

_/dev/sdX_ could for example stand for _/dev/sda_ (**not _/dev/sda1_!**)

Then create a **GRUB** config file:

```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

#### Final step

Exit out of the chroot environment by typing `exit` or pressing <kbd>Ctrl</kbd>+<kbd>d</kbd>.

Unmount all the partitions:

```
$ umount -R /mnt
```

Then type `poweroff` and remove the installation disk from the virtual machine.

## System-related Configurations

### Enable network connection

To use _pacman_ you first have to have a working internet connection by enabling NetworkManager:

```
$ systemctl start NetworkManager
$ systemctl enable NetworkManager
```

Check if you receive data from the Google Server by running this command:

```
$ ping 8.8.8.8
```

### Update the system

First things first: Update the system!

```
$ pacman -Syu
```

### `sudo` Command

```
$ pacman -S sudo
```

### Add your personal user account

```
$ useradd -m -g users -G wheel,storage,power,video,audio <your username>
$ passwd <your username>
```

#### Grant root access to our user

```
$ EDITOR=nvim visudo
```

Uncomment the following line:

```
%wheel ALL=(ALL) NOPASSWD: ALL
```

You can then login as your newly created user:

```
$ su <your username>
```

If you wish to have the default XDG directories (like Downloads, Pictures, Documents etc.) do:

```
$ sudo pacman -S xdg-user-dirs
$ xdg-user-dirs-update
```
