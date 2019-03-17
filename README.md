# ArchSetup

## Make bootable USB

Use `dd` to create bootable USB  
```
dd if=arch_installer.iso of=/dev/sdb status="progress"
```
On Windows use [Rufus](https://rufus.akeo.ie) to create installer USB.

## Boot into installer USB

1. Enable UEFI in computer firmware interface.
1. Make sure you booted into EFI mode.

   ```
   ls /sys/firmware/efi 
   ```
## Prepare hard drive

1. Identify your hard drive with

   ```
   lsblk
   ```
1. Open hard drive with `gdisk`

   ```
   gdisk /dev/sda1
   ```
1. `o` to create new GPT
1. `n` for new partition
1. `10M` for BIOS/MBR partition. Do not format. (citation needed)
1. Between `256M` & `1G` for EFI file system `ef00`
1. Make a `32M` microsoft reserved `0c01` in case you want to install Windows suddenly.
1. Make Linux root partition
1. Make Linux swap `8300` with sized 2×memory size
1. Format partitions

   ```
   mkfs.vfat -F32 /dev/sda1
   mkfs.btrfs /dev/sda3
   ```

## Installation

1. `mount /dev/sda3 /mnt` (main partition)
1. `mkdir /mnt/boot`
1. `mount /dev/sda2 /mnt/boot` (efi partition)
1. `pacstrap /mnt base base-devel arch-install-scripts b43-fwcutter btrfs-progs darkhttpd ddrescue efitools elinks exfat-utils f2fs-tools rsync fsarchiver grml-zsh-config intel-ucode ipw2100-fw ipw2200-fw lsscsi mc nfs-utils nmap networkmanager ntp pptpclient refind-efi rsync smartmontools usb_modeswitch wget wireless_tools vim wpa_supplicant`
1. `genfstab -t PARTUUID /mnt > /mnt/etc/fstab`

## Bootloader

1. `arch-chroot /mnt`
1. `bootctl install`
1. `cd /boot/loader`
1. `vim loader.conf`

   ```
   timeout 3
   default arch
   ```
1. `vim entries/arch.conf`

   ```
   title ArchLinux
   linux /vmliuz-linux
   initrd /intel-ucode.img
   initrd /initramfs-linux.img
   options root=PARTUUID=thepartuuid-without-quote rw
   ```

   `:r ! blkid` for reading it into `vim`.  
   `v` for visual mode `y` to yank and `p` to paste.

## Configuration

1. Set time zone:

   `ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime`
1. Set hardware clock:

   `hwclock --systohc`
1. Uncomment `en_US.UTF-8 UTF-8` in `/etc/locale.gen` then:

   `locale-gen`
1. `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
1. `vim /etc/hosts`
   ```
   127.0.0.1	ButterSlide
   ::1		ButterSlide
   127.0.1.1	ButterSlide.localdomain	ButterSlide
   ```
1. `echo "ButterSlide" > /etc/hostname`
1. `systemctl enable NetworkManager`
1. `systemctl enable fstrim.timer`
1. `passwd` and set root password.
1. `exit`
1. `reboot`

## Installing rEFInd bootloader

Once booted into newly installed system (not `chroot`), 
as `root`
1. `rm -rf /boot/*`
1. `pacman -S linux refind-efi`
1. `mkinitcpio -p linux` just to be safe.
1. `refind-install`

## Moar Configuration

1. `vim /etc/pacman.conf` and add these lines:

   ```
   [AUR]
   SigLeve = Never
   Server = http://repo.archlinux.fr/$arch
   ```
1. `useradd -mg users -G wheel, storage, power -s /bin/zsh borhan`
1. `passwd borhan`
1. `visudo` and uncomment:
   ```
   %wheel ALL=(ALL) ALL
   ```
   ##### Installing Nvidia
1. `sudo pacman -S linux-headers`
1. `sudo pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings`
1. `sudo vim /etc/mkinitcpio.conf` add `nvidia nvidia_modeset nvidia_uvm nvidia_drm` to MODULES
   ```
   MODULES="nvidia nvidia_modeset nvidia_uvm nvidia_drm"
   ```
1. add those modules to rEFInd bootloader config too.
1. `mkdir /etc/pacman.d/hooks`
1. `sudo vim /etc/pacman.d/hooks/nvidia.hook`
   ```
   [Trigger]
   Operation=Install
   Operation=Upgrade
   Operation=Remove
   Type=Package
   Target=nvidia

   [Action]
   Depends=mkinitcpio
   When=PostTransaction
   Exec=/usr/bin/mkinitcpio -P
   ```
