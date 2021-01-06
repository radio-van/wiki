# Contents

- [create encrypted partition](#create-encrypted-partition)
- [set bootloader options](#set-bootloader-options)
- [keys](#keys)
    - [keyfile on separate drive](#keyfile-on-separate-drive)

Tool for creating encrypted partitions.  

# create encrypted partition
* `cryptsetup luksFormat /dev/sdaX`
* `cryptsetup open /dev/sdaX cryptlvm`
* decrypted container is available at `/dev/mapper/cryptlvm`

# set bootloader options
   `options cryptdevice=UUID=<device-UUID>:cryptlvm root=/dev/MyVolGroup/root`  
       `<device-UUID>` can be checked with `lsblk -dno UUID /dev/sdaX`  
       if [keyfile](#keyfile-on-separate-drive) is used, add it to **boot options**
# keys
## keyfile on separate drive
`keyfile` can be random file, passphrase, binary

1. `cryptsetup luksAddKey <device> <keyfile>` allow to decrypt partition with a given **keyfile**

2. save **keyfile** somewhere  
    - store key between partitions
        `dd if=<keyfile> of=/dev/<device> bs=1 seek=X count=N`
        where 
        - `<device>` is third device (e.g. usb-flashdrive) to store key between/before partitions
        - `X` - offset on `<device>` (e.g. 2048 to preserve MBR header) in bytes
        - `Y` - size of key in bytes

    - recover key
        `dd if=/dev/<device> of=<keyfile> bs=1 skip=X count=N`

* setup automounting
    - for root partition edit `/boot/loader/entries/<entry>.conf`
        `options cryptdevice=UUID=...:<LVMGroup> cryptkey=<keyfile>:<offset>:<size> root=/dev/<LVMGroup>/...`
        *NOTE*: if `<keyfile>` path contains `:`, they must be escaped with `\`

    - for non-root filesystems `/etc/crypttab`
    `<Name> <UUID> <device-with-key> luks,timeout=180,keyfile-offset=Y,keyfile-size=N`

* open w/o automounting
`crypttab open <device> --key-file <device-with-key> --keyfile-offset=Y --keyfile-size=N`
