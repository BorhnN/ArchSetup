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


