# ArchLinux Luks Install

Based on [this video](https://www.youtube.com/watch?v=XNJ4oKla8B0)


## Find keymaps

```
localectl list-keymaps | grep fr
```

```
loadkeys fr
```

## Check internet connection

```
ip a
```

```
iwctl
```

To connect follow [this](https://wiki.archlinux.org/title/Iwd#iwctl)

## Package servers

```
sudo pacman -Syy reflector
reflector -c France -a 6 --sort rate --save /etc/pacman.d/mirrorlist
pacman -Syyy
```

## Partitions


Check the devices' names
```
lsblk
```

### Create partitions

```
gdisk /dev/devicename
```

Inside gdisk:
```
n
<enter_for_default>
<enter_for_default>
+200M
ef00
n
<enter_for_default>
<enter_for_default>
<enter_for_default>
<enter_for_default>
w
```

### Encrypt partitions

Encrypt partition 2 (name: partition_name2)
```
cryptsetup -y -v luksFormat /dev/partition_name2
```

cryptroot will be the name of the encrypted parition, another name can be picked
```
cryptsetup open /dev/parition_name2 cryptroot
```

### Format partitions

```
mkfs.ext4 /dev/mapper/cryptroot
mkfs.fat -F32 /dev/partition_name1
```

### Mount partitions

```
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/partition_name1 /mnt/boot
```

Check with `lsblk`

## Installing base packages

```
pacstrap /mnt base linux linux-firmware vim intel-ucode
```

## Generating file system table

```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Leaving the installer

```
arch-chroot /mnt
```

## Creating the swap file

```
fallocate -l 2GB /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Open `/etc/fstab`
Add line with vim
```
/swapfile none swap defaults 0 0
```

## Localisation

```
timedatectl list-timezones | grep Paris
ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime
hwclock --systohc
```

Open `/etc/locale.gen` and uncomment line according to what you want

```
locale-gen
echo LANG=en_US.UTF-8 >> /etc/locale.conf
echo KEYMAP=fr >> /etc/vconsole.conf
```

## Hostname

Open `/etc/hostname` and write

```
computer_name
```

Open `/etc/hosts` and write

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   computer_name.localdomain   computer_name
```

## Password for root user

```
passwd
```

## Install grub and stuff

```
pacman -S grub efibootmgr networkmanager network-manager-applet wireless_tools wpa_supplicant dialog os-prober mtools dosfstools base-devel linux-headers reflection git pulseaudio cups xdg-utils xdg-users-dirs
```

## Change mkinitcpio

Open `/etc/mkinitcpio.conf` and change the line with `HOOKS` with
```
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems keyboard fsck)
```

```
mkinitcpio -p linux
```

## Setup grub

```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Find the name of the encrypted partition UUID

```
blkid > /tmp/uuid
```

Open `/etc/defaults/grub` and change value of `GRUB_CMD_LINE_LINUX` to `cryptdevice=UUID=THE_OTHER_UUID:cryptroot root=/dev/mapper/cryptroot"`

```
grub-mkconfig -o /boot/grub/grub.cfg
```

## Enable services

```
systemctl enable NetworkManager
systemctl enable org.cups.cupsd
```

## Create users

```
useradd -mG wheel <user_name>
passwd <user_name>
EDITOR=vim visudo
```

Uncomment the line with `%wheel ALL=(ALL) ALL`

## Exit

```
exit
umount -a
reboot
```

## As new logged in user

### Connect to wifi

```
nmtui
```

### Install graphic card driver

```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

### Setup display manager

See [LightDM](https://wiki.archlinux.org/title/LightDM)

You should install `lightdm xorg-server lightdm-gtk-greeter`

```
sudo systemctl enable lightdm
```

#### Configure keyboard mapping in display manager

Create file `/etc/X11/xorg.conf.d/20-keyboard.conf` and write

```
Section "InputClass"
    Identifier "keyboard"
    MatchIsKeyboard "yes"
    Option "XkbLayout" "fr"
    Option "XkbVariant" "basic"
EndSection
```

#### Configure autologin

See [autologin](https://wiki.archlinux.org/title/LightDM#Enabling_autologin)
### Autoselection in GRUB

See [GRUB](https://wiki.archlinux.org/title/GRUB/Tips_and_tricks#Hide_GRUB_unless_the_Shift_key_is_held_down)
