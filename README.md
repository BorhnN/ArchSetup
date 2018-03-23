
ArchSetup

Make bootable USB

Use dd to create bootable USB

dd if=arch_installer.iso of=/dev/sdb status="progress"



On Windows use Rufus to create installer USB.

Boot into installer USB
1. Enable UEFI in computer firmware interface.
2. Make sure you booted into EFI mode.


ls /sys/firmware/efi



Prepare hard drive
1. Identify your hard drive with
lsblk



2. Open hard drive with gdisk 
gdisk /dev/sda1



3.  o to create new GPT
4.  n for new partition
5. Between 256M & 1G for EFI file system ef00 
6. Make a 32M microsoft reserved 0c01 in case you want to install Windows suddenly.
7. Make Linux root partition
8. Make Linux swap 8300 with sized 2×memory size
9. Format partitions
mkfs.vfat -F32 /dev/sda1
mkfs.btrfs /dev/sda3



Installation
1.  mount /dev/sda3 /mnt 
2.  mkdir /mnt/boot 
3.  mount /dev/sda1 /mnt/boot 
4.  pacstrap /mnt base base-devel arch-install-scripts b43-fwcutter btrfs-progs darkhttpd ddrescue efitools elinks exfat-utils f2fs-tools rsync fsarchiver grml-zsh-config intel-ucode ipw2100-fw ipw2200-fw lsscsi mc nfs-utils nmap ntp pptpclient refind-efi rsync smartmontools usb_modeswitch wget wireless_tools vim wpa_supplicant 
5.  genfstab -U /mnt >> /mnt/etc/fstab 

Bootloader
1.  arch-chroot /mnt 
2.  bootctl install 
3.  cd /boot/loader 
4.  vim loader.conf 
timeout 3
default arch



5.  vim entries/arch.conf 
title ArchLinux
linux /vmliuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=thepartuuid-without-quote



 :r ! blkid for reading it into vim .
 v for visual mode y to yank and p to paste.
6.  mkinitcpio -p linux 
7.  exit 
8.  reboot 

installing rEFInd bootloader

If booted into newly installed system (not chroot ),
as root 
1.  rm -rf /boot/* 
2.  pacman -S linux refind-efi 
3.  refind-install 

