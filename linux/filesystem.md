# Contents

- [encryption](#encryption)
- [folders](#folders)
    - [bin folders](#bin-folders)
    - [etc folders](#etc-folders)
- [mount](#mount)
    - [grant write permissions to user](#grant-write-permissions-to-user)
    - [allow user to mount](#allow-user-to-mount)
    - [mount options](#mount-options)
    - [mount ramfs](#mount-ramfs)
    - [mount encrypted volumes](#mount-encrypted-volumes)
- [partitions](#partitions)
    - [copy partition](#copy-partition)
    - [shrink partition](#shrink-partition)
    - [virtual partitions](#virtual-partitions)
- [swap](#swap)
    - [increase size of swapfile](#increase-size-of-swapfile)
- [ZFS](#zfs)
    - [Copy-on-write](#copy-on-write)
    - [Snapshots](#snapshots)
    - [Decompression](#decompression)
    - [Topology](#topology)
        - [device](#device)
        - [vdev](#vdev)
        - [pool](#pool)
        - [dataset](#dataset)
        - [zvol](#zvol)
    - [recommendations](#recommendations)
    - [rename pool](#rename-pool)
    - [encrypt unencrypted pool](#encrypt-unencrypted-pool)
- [tmpfs](#tmpfs)
    - [increase size](#increase-size)
- [dd](#dd)
    - [mount partition from full-disk image](#mount-partition-from-full-disk-image)
- [loop device](#loop-device)
- [find out what blocks unmounting](#find-out-what-blocks-unmounting)
- [move dataset](#move-dataset)
- [encryption](#encryption-2)
    - [change to inherit](#change-to-inherit)
- [ACL](#acl)

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

## shrink partition
* `e2fsck -f /dev/sdXY`
* `resize2fs /dev/sdXY <size>`, e.g. `30G` 
  take a note of output: `The filesystem on /dev/sdXY is now N (M) blocks long`
* `fdisk /dev/sdX` then `p`
* take a look at the partition, its **End** param is what need to be altered
* delete the partition with `d`
* create a new one with `n` with same params as the old one, but set las sector to `N*M`,
  e.g `6291456 * 4k = 25165824k` => `+25165824K` (note: **+** and **K**)
  
## virtual partitions
See [LVM](../utilities/lvm.md)

# swap
## increase size of swapfile
* `swapoff <swapfile path>`
* `fallocate -l <new size> <swapfile path>`
* `mkswap <swapfile path>`
* `swapon <swapfile path>`


# ZFS

**ZFS** keeps hashes for all data and checks on read or while doing **scrub** task to keep data consistent.

## Copy-on-write
Conventional FS modifies files in-place. CoW FS writes new copy on free space and marks old copy as
available to be overwritten by another data.
Link and unlink blocks is a single operation and even power loss during it doesn't affect consistency:
data will stay either in old state or in new one.

## Snapshots
* because of CoW it is easy to keep versions of files: FS just doesn't mark old version as "free space" until
at least one snapshot is linked with that copy.
* send/receive is very fast, because only changed data is transfered and diffs don't have to be calculated, which  
makes it signifiacantly faster than rsync.

## Decompression
Decompression is fast and efficient. Recommended to keep no more than 80% of pool occupied.

## Topology

### device
Physical device: typically SSD or HDD, but also can be hardware RAID or even a raw file

### vdev
Consists of one or more **devices** which can be combined in several ways:
* single
* mirror (of any number of devices)
* RAIDZ1, RAIDZ2, RAIDZ3 combination with 1, 2 or 3 disks for parity data
NOTE: parity data is actually distributed across __all__ disks.
NOTE: important param is `ashift`, which sets **block** size. Ideally `2^ashift = device sector size`.
      **block** smaller than **sector** dramatically decreases performance, as each **sector**
      should be readed before modify data amount equal to **block** size.
      **block** bigger that **sector** size decrease space usage efficiency, but not significantly.
      As some devices can provede false **sector** size information, better to set `ashift = 13`
* special: for log, cache, metadata

### pool
Consists of 1 or more **vdev**s. Has **no** parity, all redundancy is made on **vdev** level.
**Spare** device can be added on **pool** level to be attached to _temporary_ replace
failed one in any **vdev**. However, failed **device** in **vdev** still must be replaced,
after that **spare** becomes inactive again.

### dataset
Roughly, a directory with a set of params such as compression, quota, etc.
Mountpoint of any dataset can be anywhere.

### zvol
Virtual block device which can be formatted into any filesystem


## recommendations
* turn off `atime` per dataset
* set dataset attributes: `quotas`, `mountpoints`, `deduplication` (requires hashtable for all blocks), `compress`


## rename pool
- `zpool import old_name new_name`
- `zpool export new_name`


## encrypt unencrypted pool
Native ZFS encryption does not encrypt pools, it only encrypts filesystems.
Further, per-filesystem encryption has to be set up at the time the filesystem is created.
In the case of the pool's root filesystem, this means that encryption of the root filesystem must be set at the time the pool is created.

So to create a new pool with an encrypted root filesystem in a similar fashion to that of your existing pool, begin by destroying the new pool:

`zpool destroy <pool>`

Then re-create the pool, merging your zpool create command above with these additional options.
Note the use of capital `-O`:
```
zpool create \
    (your options from above) \
    -O encryption=on \
    -O keyformat=(whatever) \
    -O keylocation=(whatever) \
    <pool> \
    mirror /dev/gpt/diskA-serial-num /dev/gpt/diskB-serial-num
```

Finally, verify:
```
zfs get encryption,keyformat,keylocation <pool>

NAME   PROPERTY     VALUE        SOURCE
pool  encryption   aes-256-gcm  -
pool  keyformat    (whatever)   -
pool  keylocation  (whatever)   local
```


# tmpfs

## increase size
`mount -o remount,size=<X>G /tmp`


# dd

## mount partition from full-disk image
* `fdisk -lu <img>`:
    ```
    ...
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    ...
      Device Boot      Start         End      Blocks   Id  System
    sda.img1   *          56     6400000     3199972+   c  W95 FAT32 (LBA)
    ```

* calculate offset: **Sector size** * **Start** = (e.g.) `512` * `56` = `28672`

* mount using the offset:
    ```
    losetup -o 28672 /dev/loop0 <img>
    ```
* now the partition resides on /dev/loop0 and can be fscked, mounted, etc
    ```
    fsck -fv /dev/loop0
    mount /dev/loop0 /mnt
    ```
* unmount
    ```
    umount /mnt
    losetup -d /dev/loop0
    ```
    
    
# loop device
Allows to mount files containing filesystems as regular block devices. Useful for mounting disk images.
* `losetup -f` for getting first available loop device
* 
  ```
  losetup /dev/loopX <image>.img
  mount /dev/loopX <mountpoint>
  ```
  **OR**
  `mount -o loop <image>.img <mmmountpoint>`


# find out what blocks unmounting
`lsof <mountpoint>`


# move dataset
`zfs rename <old path> <new path>`
*NOTE*: this preserve encryption, if target dataset must be encrypted and source dataset is not,
better to copy data via rsync


# encryption

## change to inherit
`zfs change-key -i <path/to/dataset>`


# ACL
`setfacl` / `getfacl`
