# Contents

- [setup](#setup)
- [workaround on stack LUKS+LVM volume](#workaround-on-stack-lukslvm-volume)

# setup
* create physical volume `pvcreate /dev/mapper/cryptlvm` 
* create virtual volume group `vgcreate MyVolGroup /dev/mapper/cryptlvm`  
* create partitions
```
  lvcreate -L 8G MyVolGroup -n swap
  lvcreate -L 32G MyVolGroup -n root
  lvcreate -l 100%FREE MyVolGroup -n home
```
* make filesystems
```
  mkfs.ext4 /dev/MyVolGroup/root
  mkfs.ext4 /dev/MyVolGroup/home
  mkswap /dev/MyVolGroup/swap
```

# workaround on stack LUKS+LVM volume
sympthoms: `device <device> still in use`  

* `vgchange -a n <name>` fix activation of `<name> VolumeGroup`  
  `vgdisplay` shows all `VolumeGroup`s
* `cryptsetup close <device>` can be used afterwards
