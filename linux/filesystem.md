# Contents

- [encryption](#encryption)
- [folders](#folders)
    - [bin folders](#folders#bin folders)
    - [etc folders](#folders#etc folders)
- [mount](#mount)
    - [grant write permissions to user](#mount#grant write permissions to user)
    - [allow user to mount](#mount#allow user to mount)
    - [mount options](#mount#mount options)
    - [mount ramfs](#mount#mount ramfs)
    - [mount encrypted volumes](#mount#mount encrypted volumes)
- [partitions](#partitions)
    - [copy partition](#partitions#copy partition)
    - [virtual partitions](#partitions#virtual partitions)
- [swap](#swap)
    - [increase size of swapfile](#swap#increase size of swapfile)

# encryption
See [LUKS](../utilities/luks.md)

# folders
## bin folders
`/bin` ::
    system commands needed when no FS are mounted (e.g. single-user mode), like *cat*, *cp*, *dd*, *rm*, ...
    _managed by Packet Manager_
`/sbin` ::
    system administration utilities, like *fdisk*, *ifconfig*, *mkfs*, *reboot*, ...
`/usr/bin` ::
    commands, typical for all company's servers (i.e. this dir is the same for all servers, even if OS are different)
    also, links to interpreters (*Python*, *Perl*)
    _managed by Packet Manager_
`/usr/sbin` ::
    the same, but contains administrative utilities
`/usr/local/bin`, `/usr/local/sbin` ::
    OS-depended, hardware-depended utilities
    _not managed by Packet Manager_, *make install* _goes here_
`/home/$USER/bin` ::
    utilities for particular user, should be synced between hosts with the same user
    
## etc folders
`/etc/default` contains parameters that user is likely to change. Those modifications are preserved across package uprades.
In `System V` for each service in `/etc/init.d/<service>` default file from `/etc/default/<service>` is being sourced.
The purpose is to provide extra options on starting the service or override some aspects of the service's startup.

# mount
## grant write permissions to user
`sudo shmod ugo+wx <mountpoint>`

## allow user to mount
`/etc/fstab`
add `users` option to mount options

## mount options
* atime
    * `strictatime` updates access time of every file (only useful for servers)
    * `noatime` disables writing file access times every time you read a file (useful in most cases, but should not be used for apps like *Mutt* which need to know last time file was read
    * `nodiratime` disables writing file access times for directories
    * `relatime` updates atime only if previous was earlier than current modify/change time
    * `lazytime` on-disk timestamps are updated only during file changes/sync/24 hours/etc. The rest of time timestamps are stored in memory.
* auto
    * `auto` allows mount with `mount -a`
    * `noauto` requires explicit mount
* `exec` permit execution of binaries
* `async` async I/O

* `owner`, `group` allows device to be mounted by the user/group with is same as device's
* `user`, `users` allows to specific user (or all users) to mount this FS
all 4 above options implie `nosuid, nodev`

* `nofail` skips errors if device doesn't exests (useful for mount external drives during boot)
* `remount` attempt to remount an already-mounted FS

* `x-systemd.automount` mount not at a boot time, but at access moment
* `x-systemd.mount-timeout=N` timeout if disk is not accessible

## mount ramfs
`mount ramfs <mountpoint> -t ramfs`

## mount encrypted volumes
`/etc/crypttab`
`<name> UUID=<uuid of /dev/sdXY aka LUKS partition>    <password>    luks,timeout=N,[key options]`
where
* `<password>` is a path to keyfile, e.g. by physical position `/dev/disk/by-path/pci-...`
* `key options` could be key offset/size `keyfile-offset=<bytes>,keyfile-size=<bytes>`

# partitions
## copy partition
* `dd if=/dev/sdXN of=/dev/sdYN` more about [dd](../utilities/dd.md)
* `resize2fs /dev/sdYN` in case if destination drive is bigger
* `tune2fs -U random /dev/sdYN` to generate uniq `UUID` for copied partition

## virtual partitions
See [LVM](../utilities/lvm.md)

# swap
## increase size of swapfile
* `swapoff <swapfile path>`
* `fallocate -l <new size> <swapfile path>`
* `mkswap <swapfile path>`
* `swapon <swapfile path>`
