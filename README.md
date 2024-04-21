# Arch_Btrfs_SystemD
Install Guide to install Arch on BTRFS and Systemd booting

This guide is just for my HP setup

First test network
I am using my phone usb tether 

ip a will show if there is a addresses assigned.

fdisk -l 
this will show all the drives

lspci -vv | more , I use this to find the drive link speed
pacman -S hwinfo then hwinfo --disk | more 
this to correnspond those two things


First check
/dev/sda is the 2.5" sata SSD
/dev/nvme0n1 is a 2TB on the Wifi m.2 slot , so this one is slowers
/dev/nvme1n1 is a 2TB on the fasters m.2 slot.

Hard Drive Partitions

/dev/sda1 this is the efi boot partition at 512M
/dev/sda2 this is the root partition at 128GB 
/dev/nvme1n1 this has the swap parition at 32GB

Format Drives

mkfs.vfat -F32 -n EFI /dev/sda1
mkfs.btrfs -L ROOT /dev/sda2
mkswap /dev/nvme1n1p1

swapon

create the BTRFS Sub Volunes

# mount /dev/sda2 /mnt
# btrfs sub create /mnt/@
# btrfs sub create /mnt/@tmp
# btrfs sub create /mnt/@log
# btrfs sub create /mnt/@pkg
# btrfs sub create /mnt/@snapshots
# umount /mnt

Mount the sub volumes:

# mount -o noatime,nodiratime,compress=zstd:2,space_cache=v2,ssd,subvol=@ /dev/sda2 /mnt
# mkdir -p /mnt/{boot,var/log,var/cache/pacman/pkg,.snapshots,btrfs,tmp}
# mount -o noatime,nodiratime,compress=zstd:5,space_cache=v2,ssd,subvol=@pkg /dev/sda2 /mnt/var/cache/pacman/pkg
# mount -o noatime,nodiratime,space_cache=v2,ssd,compress=zstd:5,subvol=@log /dev/sda2 /mnt/var/log
# mount -o noatime,nodiratime,space_cache=v2,ssd,compress=zstd:6,subvol=@tmp /dev/sda2 /mnt/tmp
# mount -o noatime,nodiratime,compress=zstd:4,space_cache=v2,ssd,subvol=@snapshots /dev/sda2 /mnt/.snapshots

mount the efi partition

mount /dev/sda1 /mnt/boot

install the base system

pacstrap /mnt linux-zen linux-zen-headers linux-firmware base btrfs-progs amd-ucode nano networkmanager

arch-chroot /mnt/

echo HP-Arch > /etc/hostname

echo LANG=en_US.UTF-8 > /etc/locale.conf
/etc/hosts
nano -w /etc/locale.gen
uncomment 
en_US.UTF-8

locale-gen

ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime

hwclock --systohc

/etc/hosts

127.0.0.1	HP-Arch.localdomain	HP-Arch
::1		localhost.localdomain	localhost

passwd

Initramfs
nano -w /etc/mkinitcpio.conf 

HOOKS=(base keyboard udev autodetect modconf block keymap btrfs filesystems)

mkinitcpio -P linux

Boot Manager

bootctl --path=/boot install

blkid -s UUID -o value /dev/sda2

nano -w /boot/loader/entries/arch.conf

title Arch Linux
linux /vmlinuz-linux-zen
initrd /amd-ucode.img
initrd /initramfs-linux-zen.img
options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw

nano -w /boot/loader/loader.conf

default  arch.conf
timeout  4
console-mode max
editor   no

systemctl enable NetworkManager

# exit
# umount -R /mnt
# reboot








