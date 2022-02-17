# Contents

- [setup](#setup)
- [workaround on stack LUKS+LVM volume](#workaround on stack LUKS+LVM volume)
- [mount volume group with the same name](#mount volume group with the same name)
- [copy logical volume](#copy logical volume)
- [resize logical volume](#resize logical volume)
    - [shrink](#resize logical volume#shrink)
    - [expand](#resize logical volume#expand)

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

# mount volume group with the same name
* `lvdisplay / vgdisplay` to figure out correct group
* `vgrename <UUID> <new_name>`
* `vgchange -ay` activate renamed group
* `mount /dev/<vg group>/<partition> <mountpoint>`

# copy logical volume
* a snapshot of the targeting logical volume  
    `lvcreate --snapshot --name <snapshot-name> --size <size> /dev/<source-volume-group>/<source-logical-volume>`  
    *NOTE:* Size of the snapshot must be enough only to capture changes during snapshot period

* an image copy of the snapshot content
    `dd if=/dev/<source-volume-group>/<snapshot-name> of=<backup.img>`

* a new logical volume of **enough size** in the *targeting (new)* volume group
    `lvcreate --name <new-volume-name> --size <size> <new-volume-group-name>`

* write data to the *new* logical volume from the image backup
    `dd if=<backup.img> of=/dev/<new-volume-group>/<new-logical-volume>`

* delete the *snapshot* and image *backup* using `lvremove` and `rm` respectively

# resize logical volume
* `vgchange -ay` activate volumes
* any mounted volumes should be unmounted (for *root* volumes LiveImage can be used)
* `e2fsck -fy /dev/<VolumeGroup>/<partition>` check filesystem

## shrink
* `resize2fs /dev/<VolumeGroup>/<partition> <new size>` shrink filesystem
* `lvreduce -L <reduce to size> /dev/<VolumeGroup>/<name>`  
  OR `lvreduce -L -<reduce by size> /dev/<VolumeGroup>/<name>` shrink volume
* `resize2fs /dev/<VolumeGroup>/<partition>` expand filesystem to the whole volume

## expand
* space for expanding can be gained by [shrinking](##shrink) another volume
* `lvextend -l +100%FREE /dev/<VolumeGroup>/<partition>`  
  OR `lvextend -L <extend to size> /dev/<VolumeGroup>/<partition>`  
  OR `lvextend -L +<extend by size> /dev/<VolumeGroup>/<partition>`  
* `resize2fs /dev/<VolumeGroup>/<partition>` expand filesystem to the whole volume
