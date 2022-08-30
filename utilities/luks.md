# Contents

- [create encrypted partition](#create encrypted partition)
- [set bootloader options](#set bootloader options)
- [keys](#keys)
    - [generate random key](#keys#generate random key)
    - [keyfile on separate drive](#keys#keyfile on separate drive)
        - [keyfile between partitions](#keys#keyfile on separate drive#keyfile between partitions)
        - [keyfile as a file](#keys#keyfile on separate drive#keyfile as a file)
    - [use TPM](#keys#use TPM)
    - [potentially useful stuff](#keys#potentially useful stuff)
- [LUKS container on remote host](#LUKS container on remote host)

Tool for creating encrypted partitions.  

# create encrypted partition
* `cryptsetup luksFormat /dev/sdaX`
* `cryptsetup open /dev/sdaX <crypt_vol>`
* decrypted partition at `/dev/mapper/<crypt_vol>` can be formatted and used as usual
  e.g. `mkfs.ext4 /dev/mapper/...`

# set bootloader options
   `options cryptdevice=UUID=<device-UUID>:cryptlvm root=/dev/MyVolGroup/root`  
       `<device-UUID>` can be checked with `lsblk -dno UUID /dev/sdaX`  
       if [keyfile](#keyfile-on-separate-drive) is used, add it to **boot options**
# keys
## generate random key
* `dd if=/dev/urandom of=<keyfile> bs=512 count=8`
* `tr -dc A-Za-z0-9 </dev/urandom | head -c 64 ; echo ''`

## keyfile on separate drive
`keyfile` can be random file, passphrase, binary

`cryptsetup luksAddKey <device> <keyfile>` allow to decrypt partition with a given **keyfile**

### keyfile between partitions
`dd if=<keyfile> of=/dev/<device> bs=1 seek=X count=N`  
where:
- `<device>` is third device (e.g. usb-flashdrive) to store key between/before partitions
- `X` - offset on `<device>` (e.g. 2048 to preserve MBR header) in bytes
- `Y` - size of key in bytes

**TIP:** recover key:
  `dd if=/dev/<device> of=<keyfile> bs=1 skip=X count=N`

* for root partition edit `/boot/loader/entries/<entry>.conf`
    `options cryptdevice=UUID=...:<LVMGroup> cryptkey=<keyfile>:<offset>:<size> root=/dev/<LVMGroup>/...`
    *NOTE*: if `<keyfile>` path contains `:`, they must be escaped with `\`

* for non-root filesystems `/etc/crypttab`
  `<Name> <UUID> <device-with-key> luks,timeout=180,keyfile-offset=Y,keyfile-size=N`
    
* open w/o automounting
  `crypttab open <device> --key-file <device-with-key> --keyfile-offset=Y --keyfile-size=N`
    
### keyfile as a file
* for root partition edit `/boot/loader/entries/<entry>.conf`
  `options cryptdevice=UUID=...:<LVMGroup> cryptkey=UUID=...:auto:<keyfile name> root=/dev/<LVMGroup>/...`
        *NOTE*: if `<keyfile>` path contains `:`, they must be escaped with `\`
* add `vfat` to `mkinitcpio.conf`'s `MODULES=(...)`

* open w/o automounting
  `crypttab open <device> --key-file <path/to/keyfile>`

## use TPM
`TPM` is a secure chip that is present on most modern PCs.  
* check support: `journalctl -k --grep=tpm`
* install:
    * `libpwquality` provides `pwmake`
    * `tpm2-tools`
    * `clevis` for binding **LUKS** partitions
* `clevis luks bind -d /dev/sdX tpm2 '{}'`  
  `{}` is configuration, e.g. `'{"pcr_ids":"1,7"}'` will check UEFI/Secure Boot consistency
* enable `clevis-luks-askpass.path` **systemd** unit for  
  automatic decryption via `/etc/crypttab` at boot
  
Manual decryption: `clevis luks unlock -d /dev/sdX -n MAPPER`

## potentially useful stuff
* `decrypt_keyctl` allows to use script for **LUKS** key generation (?)
* `booster` is an alternative **initramfs** generator
* `secure boot` can be used with **TPM2**


# LUKS container on remote host
* `sshfs -o allow_root <host>:<path> <mount point>`
* `truncate -s <container size> <mount point>/<container file>`
* `cryptsetup luksFormat <mount point>/<container file>`
* `cryptsetup luksOpen <mount point>/<container file> <name>`
* `mkfs.ext4 -m0 -E lazy_itable_init=0,lazy_journal_init=0 /dev/mapper/<name>`
