#### UEFI or BIOS?

Run the following command:

```
$ ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. Else you have to use BIOS mode.

#### GET FAST MIRROR 

Run the following command:

```
$ pacman -S y reflector
$ reflector -c Vietnam -c Singapore -c Janpan -c India -a 12 --sort rate --save /etc/pacman.d/mirrorlist
```

#### Partition disk

Run the following command to see disk:

```
$ lsblk
$ cfdisk /dev/sda
```

##### Partition formatting

```
$ mkfs.ext4 /dev/root_partition  => root
$ mkswap /dev/swap_partition => swap
$ mkfs.fat -F 32 /dev/efi_system_partition => boot
```

##### Mounting the file system

```
$ mount /dev/root_partition /mnt         => mount root
$ mkdir /mnt/home      => create home directory
$ mount /dev/home_partition /mnt/home    => mount home
$ mkdir /mnt/boot
$ mount /dev/efi_system_partition /mnt/boot  => mount boot
$ swapon /dev/swap_partition
```

### Package install

For a minimal system download and install these packages:

```
$ pacstrap -K /mnt base base-devel linux linux-firmware linux-headers dhcpcd networkmanager network-manager-applet git nano
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
$ ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
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

#### Network configurations

Create _/etc/hostname_ and type any name you wish as your hostname:

```
arch
```

Edit _/etc/hosts_ like this:

```
127.0.0.1 localhost
::1 localhost
127.0.1.1 arch.local  arch
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

### Add your personal user account

```
$ useradd -m <your username>
$ passwd <your username>
$ usermod -aG wheel,storage,power,video,audio,optical <your username>
```

#### Grant root access to our user

```
$ EDITOR=nvim visudo
```

Uncomment the following line:

```
%wheel ALL=(ALL) ALL
```

### Enable network connection

To use _pacman_ you first have to have a working internet connection by enabling NetworkManager:

```
$ systemctl start NetworkManager
$ systemctl enable NetworkManager
```

#### Bootloader

##### UEFI

Install `grub` and `efibootmgr`:

```
$ pacman -S grub efibootmgr
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

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



