# ArchSetup

## Make bootable USB

use `dd` to create bootable USB

```
dd if=arch_installer.iso of=/dev/sdb 
```
on Windows use [Rufus](https://rufus.akeo.ie) to create installer USB.

## Boot into installer USB

1. enable UEFI in computer firmware interface.
2. make sure you booted into EFI mode.
```
ls /sys/firmware/efi
```

## Prepare hard drive

1. identify your hard drive with
```
lsblk
```
2. open hard drive with `gdisk`
```
gdisk /dev/sda1
```
3. `o` to create new GPT
4. `n` for new partition
5. between `256M` & `1G` for EFI file system `ef00`
6. make a `32M` microsoft reserved `0c01` in case you want to install Windows suddenly.
7. make Linux root partition
8. make Linux swap `8300` with sized 2×memory size
9. format partitions
```
mkfs.vfat -F32 /dev/sda1
mkfs.btrfs /dev/sda3
```

## Installation

1. `mount /dev/sda3 /mnt`
2. `mkdir /mnt/boot`
3. `mount /dev/sda1 /mnt/boot`
4. `pacstrap /mnt base base-devel vim`

## Bootloader

1. `arch-chroot /mnt`
2. `bootctl install`
3. `cd /boot/loader`
4. `vim loader.conf`
```
timeout 5
default arch
```
6. `vim entries/arch.conf`
```
title ArchLinux
linux /vmliuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=thepartuuid-without-quote
```

8. `mkinitcpio -p linux`
9. `exit`
9. `reboot`
