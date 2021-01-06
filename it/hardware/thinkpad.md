# fwupd

## UEFI partition
To solve `WARNING: UEFI ESP partition not detected or configured`  
- install `UDisks`
or
- add `OverrideESPMountPoint=/boot/EFI` to `/etc/fwupd/uefi.conf`

