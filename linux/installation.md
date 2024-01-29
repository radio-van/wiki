# Contents

    - [base](#base)
        - [pre-installation](#pre-installation)
        - [partition](#partition)
            - [encryption](#encryption)
                - [LVM on LUKS](#lvm-on-luks)
        - [install base system](#install-base-system)
        - [generate fstab](#generate-fstab)
        - [chroot](#chroot)
        - [adjust time](#adjust-time)
        - [locale](#locale)
        - [hostname](#hostname)
        - [initramfs](#initramfs)
        - [bootloader](#bootloader)
            - [systemd-boot](#systemd-boot)
        - [pacman](#pacman)
        - [users](#users)
            - [sudo](#sudo)
    - [graphics](#graphics)
        - [fonts](#fonts)
        - [xorg](#xorg)
            - [configuration files](#configuration-files)
            - [known issues](#known-issues)
                - [Fatal server error: (EE) AddScreen/ScreenInit](#fatal-server-error-ee-addscreenscreeninit)
        - [background](#background)
        - [window manager](#window-manager)
            - [i3](#i3)
    - [login](#login)
        - [auto login](#auto-login)
            - [manual](#manual)
            - [using display manager](#using-display-manager)
    - [configs](#configs)
        - [dotfiles](#dotfiles)
    - [power management](#power-management)
        - [sleep and hibernation](#sleep-and-hibernation)
    - [additional hardware](#additional-hardware)
        - [Lenovo's bluetooth](#lenovos-bluetooth)

## base
### pre-installation
* check if EFI is used `ls /sys/firmware/efi/efivars`
* set ntp-time `timedatectl set-ntp true`

### partition
default scheme is
- `/boot` if EFI is used
- `/root` ~20-30Gb, depends on amount of software
- `/home` for user's stuff
- `swap` ~ double? of RAM for suspend to disk

use `fdisk` for partitioning, set partition type to `Linux filesystem`
use `mkfs.ext4` for create filesystem

if installing on system with existing `EFI system partition` (e.g. on Apple computer), it could be mounted and used

more about [partitions](filesystem.md#partitions)  

#### encryption
##### LVM on LUKS
```
   +----------------------------+
   | boot | luks [encrypted]... |
   +------+----------+----------+
   | .... | lvm-part | lvm-part |
   +------+---------------------+
```

1. [create encrypted partition](../utilities/luks.md#create-encrypted-partition)
2. [create LVM partitions](../utilities/lvm.md#setup)
3. mount partitions
   - mount `/boot` (existing, or create new one) `mount /dev/sdbN /mnt/boot`
   - mount the rest
    ```
      mount /dev/MyVolGroup/root /mnt
      mkdir /mnt/home
      mount /dev/MyVolGroup/home /mnt/home
      swapon /dev/MyVolGroup/swap
    ```
4. set `mkinitcpio` hooks, install `lvm2` package
   `HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt lvm2 filesystems fsck)`
   *ATTENTION* follow the order of hooks!
5. [set bootloader option](../utilities/luks.md#set-bootloader-options)  
   if `systemd-boot` is used, config file is `<EFI System Partition>/loader/entries/<name>.conf`, `<name>` must be populated in `<EFI System Partition>/loader/loader.conf`

### install base system
`pacstrap /mnt base linux linux-firmware <any additional packages>`, e.g. `lvm2` for `LVM`

### generate fstab
`genfstab -U /mnt >> /mnt/etc/fstab`

### chroot
`arch-chroot /mnt`

### adjust time
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
`hwclock --systohc`

### locale
Uncomment `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`
`locale-gen`
`echo LANG=en_US.UTF-8 >> /etc/locale.conf`

### hostname
`echo hostname >> /etc/hostname`
edit `/etc/hosts`
```
    127.0.0.1	localhost
    ::1		    localhost
    127.0.1.1	<hostname>.localdomain	<hostname>
```

### initramfs
if any changes (e.g. hooks) are made in `/etc/mkinitcpio.conf` (e.g. `LVM` or `dm-crypt`), creating new initramfs is required
`mkinitcpio -P`

force rebuild:  
`mkinitcpio -c /etc/mkinitcpio.conf -g /boot/initramfs-linux.img -k <kernel version>`  
*NOTE:* `<kernel version>` can be looked up at `/usr/lib/modules`  

### bootloader
#### systemd-boot
installation: `bootctl install`
* `esp/loader/loader.conf` (`esp` is `EFI System Partition`)
```
    default  arch
    timeout  4
    console-mode max
    editor   no
```
* `esp/loader/entries/*.conf`
```
    title   Arch Linux
    linux   /vmlinuz-linux
    initrd  /initramfs-linux.img
    options root=LABEL=arch_os rw
```
if `LUKS` and `LVM` are used, change `options` according to [[#LVM on LUKS|LVM and LUKS]]

### pacman
see [packages](packages.md)

### users
`useradd -m -G <group> <user>`
- `-m` will create home folder
- `-G` will add to group (e.g. `wheel`)
#### sudo
* add user to `sudo` (Debian-based distros) or `wheel` group `usermod -aG wheel` (`-a` preserves previous groups)
* edit sudoers file with `visudo` add/uncomment `%wheel ALL=(ALL) ALL`


## graphics
### fonts
* `terminus-font` - font for console
* `ttf-fira-code` - nice font for most cases, has ligatures, **no italic**
* `noto-fonts-emoji` - emoji support, e.g. in Discord
* `nerd-fonts-jetbrains-mono` - patched `jetbrains-mono` font with glyph collection (font awesome, weather, material, etc)

list fonts: `fc-list`  
check wich is used: `fc-match monospace`

### xorg
* install `xorg-server`
* check video card `lspci | grep -e VGA -e 3D`
* install appropriate driver
    * search `pacman -Ss xf86-video`
| Brand  | Type        | Driver               |
| AMD    | Open source | `xf86-video-amdgpu`  |
| \/     | \/          | `xf86-video-ati`     |
| Intel  | Open source | `xf86-video-intel`   |
| NVIDIA | Open source | `xf86-video-nouveau` |
| \/     | Proprietary | `nvidia`             |
| \/     | \/          | `nvidia-390xx`       |

* install `xorg-xinit`
* install *window manager* e.g. `i3-wm`
* edit `~/.xinitrc` to run *window manager* automatically
* install fonts e.g. `ttf-droid`, hint: use `fc-list` to list installed fonts
* autostart: add to `~/.bash_profile`
  ```
      if systemctl -q is-active graphical.target && [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
          exec startx
      fi
  ```
  or
  ```
     if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
         exec startx
     fi
  ```
#### configuration files
* `~/.xinitrc` to run Window Manager, e.g. `exec i3` (add `&` to run smth in background), also sources `.xprofile`
* `~/.xprofile` to run anything before Window manager, e.g. compositor `xcompmgr &`, used by login*?* managers

#### known issues
##### Fatal server error: (EE) AddScreen/ScreenInit
Use [[https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start|Early KMS start]]
* add gpu (`amdgpu`, `radeon`, ...) to `MODULES=(...)` section of `/etc/mkinitcpio.conf`
* `mkinitcpio -p linux`

### background
`feh --bg-scale /path/to/image`
### window manager
#### i3
useful additions:
  - `dmenu` dynamic menu to launch scripts and programs

## login
### auto login
#### manual
1. override default service
    `/etc/systemd/system/getty@tty1.service.d/override.conf`
    ```
       [Service]
       ExecStart
       ExecStart=-/sbin/agetty -o '-p -- <username>' --noclear --skip-login - $TERM
 
    ```
*OR*
2. create additional service
    * copy default service `cp /usr/lib/systemd/system/getty@.service /etc/systemd/system/autologin@.service`
    * create symlink in place of original one `ln -s /etc/systemd/system/autologin@.service /etc/systemd/system/getty.target.wants/getty@tty1.service`
    * modify `ExecStart=-/sbin/agetty -a USERNAME %I 38400`
    * apply 
    ```
      systemctl daemon-reload
      systemctl disable getty@tty1
      systemctl enable autologin@tty1
      systemctl start autologin@tty1
    ```
    
#### using display manager
[[https://github.com/spanezz/nodm|nodm]]

## configs
### dotfiles
Repository with workdir in *home* folder.
Initial:
```
    git init --bare $HOME/.dotfiles
    echo "alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> $HOME/.bashrc
    source .bashrc
    dotfiles config --local status.showUntrackedFiles no
```
Sync existing:
```
    echo "alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'" >> $HOME/.bashrc
    source .bashrc
    echo ".dotfiles" >> .gitignore
    git clone --bare <git-repo-url> $HOME/.dotfiles
    dotfiles checkout
    dotfiles config --local status.showUntrackedFiles no
```

## power management
### sleep and hibernation
* *sleep* aka *suspend-to-ram* writes state to RAM, significantly decrease power consumption
* *hibernate* aka *suspend-to-swap* writes state to *swap* partition/file and powers off the machine
* *hybrid-sleep* combines both, state is restored from RAM, in case of power loss - from swap

to enable resuming from swap, `resume` hook must be added to `initramfs` i.e. `/etc/mkinitcpio.conf`<br>
after `lvm2` and `encrypt` (if they are used) and after*?* `filesystem`
then, rebuild with `mkinitcpio -P`

also appropriate *kernel parameters* must be added.<br>
if `systemd-bootd` is used, then edit `/boot/loader/entities/<entity>.conf`<br>
and add `resume=<path to swap>` e.g. `resume=/dev/<LVM group>/swap`

commands are:
* `systemctl suspend`
* `systemctl hibernate`
* `systemctl hybrid-sleep`

for security install `xss-lock` and add it to autostart (e.g. `.bash_profile`)
`xss-lock -- <external locker> &` e.g. `i3locks`
it will lock system after sleep/hibernation

## additional hardware
### Lenovo's bluetooth
```
  pacman -S bluez bluez-utils
  modprobe btusb
  systemctl enable --now bluetooth
```
