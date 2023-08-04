# Arch Linux Everforest Ricing

This is a straight-forward guide for ricing Arch Linux with the [Everforest](https://github.com/sainnhe/everforest) colorscheme theme.

I use a minimal install of Arch Linux on UTM (QEMU virtual machine) on a Macbook Air M1 (2020). This guide should also work with the new Macbook Air M2 and the Pro versions.

Also in the directory `asus` you will find configurations for Arch Linux on bare metal. The configurations for your system may vary.

ℹ️ **If you already have a running Arch system with the necessary packets installed, you can go to the [Quick Ricing](#quick-ricing) section.**

## What's the final result?

Your Arch system should look something like this using my config files:

![Arch Everforest Rice](screenshot-20230618-172210Z-selected.png)

⚠️ _Caution:_ If you are installing Arch on a virtual machine you won't have the blur effect like in the image above because there is no hardware acceleration.

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

ℹ️ If you are installing Arch Linux on a computer with **ARM architecture** add the following to the above `pacstrap` command:

```
archlinuxarm-keyring
```

⚠️ If you get errors due to key then do the following:

1. Initialize _pacman_ keys and populate them:

```
pacman-key --init
pacman-key --populate
```

2. Synchronize Arch keyring:

```
archlinux-keyring-wkd-sync
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

### Install AUR package manager

To install [yay](https://github.com/Jguer/yay):

```
$ sudo pacman -S git
$ mkdir aur
$ cd aur
$ git clone https://aur.archlinux.org/yay.git
$ cd yay
$ makepkg -si
```

### Guest tools

#### SPICE support on guest (for UTM)

This will enhance graphics and improve support for multiple monitors or clipboard sharing.

```
$ sudo pacman -S spice-vdagent xf86-video-qxl
```

#### Guest additions (for VirtualBox)

This will enhance graphics and improve support for multiple monitors or clipboard sharing.

```
$ sudo pacman -S virtualbox-guest-utils
```

### Sound

```
$ sudo pacman -S pulseaudio
$ sudo pacman -S alsa-utils alsa-plugins
$ sudo pacman -S pavucontrol
```

PulseAudio Applet:

```
$ yay -S pa-applet-git
```

### Network

```
$ sudo pacman -S openssh
$ sudo pacman -S iw wpa_supplicant
```

NetworkManager Applet:

```
$ sudo pacman -S network-manager-applet
```

Enable SSH, DHCP and NM:

```
$ sudo systemctl enable sshd
$ sudo systemctl enable dhcpcd
```

### Bluetooth

```
$ sudo pacman -S bluez bluez-utils blueman
$ sudo systemctl enable bluetooth
```

### Pacman

To beautify Pacman use:

```
$ sudo nvim /etc/pacman.conf
```

Uncomment `Color` and add below it `ILoveCandy`.

ℹ️ If you have a good internet connection, you can uncomment the option `ParallelDownloads = 5`.

### Enable SSD Trim

```
$ sudo systemctl enable fstrim.timer
```

### Enable Time Synchronization

```
$ sudo pacman -S ntp
```

```
$ sudo systemctl enable ntpd
```

Then enable NTP:

```
$ timedatectl set-ntp true
```

## Graphical User Interface (GUI) Settings

### Xorg

```
$ sudo pacman -S xorg-server xorg-apps xorg-xinit xclip xdotool
```

### i3

```
$ sudo pacman -S i3
```

After installing X and i3 create a file **.xinitrc** in your home directory with the following content:

```
exec i3
```

This way you can start the X server with the command `startx`.

### Drivers

If running outside UMT we need drivers:

**Intel**:

```
sudo pacman -S xf86-video-intel xf86-video-qxl intel-media-driver mesa
```

**NVIDIA**:

```
sudo pacman -S xf86-video-nouveau nvidia libva-mesa-driver
```

### Compositor

```
$ sudo pacman -S picom
```

### Font

```
$ sudo pacman -S noto-fonts noto-fonts-emoji ttf-firacode-nerd
```

To support Asian letters:

```
$ sudo pacman -S noto-fonts-cjk
```

### Shell

```
$ sudo pacman -S zsh
```

Change default shell to zsh:

```
$ chsh -s $(which zsh)
```

### Terminal

```
$ sudo pacman -S rxvt-unicode alacritty kitty
```

### Editor

The editor should already be installed after running the _pacstrap_ command in the installation process. You can use other editors like _nano_ too.

```
$ sudo pacman -S neovim
```

### Switcher

```
$ sudo pacman -S dmenu rofi
```

Some additional stuff for rofi:

```
$ sudo pacman -S rofi-emoji rofi-calc
```

### Status Bar

```
$ sudo pacman -S polybar
```

### File Manager

```
$ sudo pacman -S ranger
```

For previews we also need **feh** and **Überzug**:

```
$ sudo pacman -S feh ueberzug
```

### Browser

```
$ sudo pacman -S firefox
```

### Media Player

```
$ sudo pacman -S vlc
```

### PDF Viewer

```
$ sudo pacman -S zathura zathura-pdf-mupdf
```

### Wallpapers

Check these amazing wallpapers that harmonize with the Everforest theme: https://github.com/Apeiros-46B/everforest-walls

### Other Tools

#### CLI utilities

```
$ sudo pacman -S tldr fzf tar gzip htop neofetch
```

- _tldr_: Commands cheat sheet
- _fzf_: Fuzzy finder
- _tar_: Enzipping/Unzipping
- _gzip_: Enzipping/Unzipping
- _htop_: CLI task manager
- _neofetch_: System information

#### Alternatives to traditional commands

```
$ sudo pacman -S fd ripgrep bat lsd
```

- _fd_: Alternative to _find_ command
- _ripgrep_: Alternative to _grep_ command
- _bat_: Alternative to _cat_ command
- _lsd_: Alternative to _ls_ command

#### GUI utilities

```
$ sudo pacman -S maim
```

- _maim_: Screenshot utiliy

#### Languages, protocols, shells

```
$ sudo pacman -S codespell go luarocks ruby rubygems composer php nodejs npm yarn python python-pip jre-openjdk jdk-openjdk julia wget curl
```

```
$ yay -S powershell-bin
```

These programming languages, protocols or shells are not that important to install if you don't need them. I installed all of them to avoid warnings in the _mason.nvim report_ when running the `:checkhealth` command in Neovim.

### Reboot

When done installing the necessary packages, run the `reboot` command.

## Quick Ricing

You can either clone the repository and move the files manually to your _~/.config_ directory or you could use the installer:

```
$ git clone https://github.com/3rfaan/dotfiles.git ~/Downloads/dotfiles && cd ~/Downloads/dotfiles
```

Then run the _install.sh_ script:

```
$ ./install.sh
```

If you are on **VirtualBox**, run this command instead:

```
$ ./install.sh vbox
```

If you are on **UTM (or QEMU)**, run this command instead:

```
$ ./install.sh utm
```

If you want to copy the config files manually you also have to install these packages manually:

- [Packer](https://github.com/wbthomason/packer.nvim)
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
- [ranger_devicons](https://github.com/alexanderjeurissen/ranger_devicons)
1