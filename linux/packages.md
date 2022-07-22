# Contents

- [PACMAN](#PACMAN)
    - [init](#PACMAN#init)
    - [conflicts on sysupgrade](#PACMAN#conflicts on sysupgrade)
    - [corrupted packages](#PACMAN#corrupted packages)
    - [partial upgrade](#PACMAN#partial upgrade)
    - [installation log](#PACMAN#installation log)
    - [downgrade package](#PACMAN#downgrade package)
    - [save to list](#PACMAN#save to list)
    - [install from list](#PACMAN#install from list)
    - [upgrade](#PACMAN#upgrade)
    - [delete orphans](#PACMAN#delete orphans)
    - [delete cache](#PACMAN#delete cache)
    - [AUR packages](#PACMAN#AUR packages)

# PACMAN

## init
* initialize keys `pacman-key --init`
* populate keys `pacman-key --populate <distro>`, `distro` can be found at `/usr/share/pacman/keyrings/`

## conflicts on sysupgrade
```
  error: failed to commit transaction (conflicting files)
  <package> <filepath> exists in filesystem`
```

* check which package owns the file `pacman -Qo <filepath>`
  if it is another package, then it is a bug
* force overwrite file `pacman -S <package> --overwrite <filepath>`

## corrupted packages
**error** `... is corrupted (... PGP sign).`  
Update keyring: `pacman -Sy archlinux-keyring`

**NOTE**: always follow this by `pacman -Syu` as [partial upgrades](##partial upgrade) is a bad idea

## partial upgrade
**NEVER** use `pacman -Sy / pacman -Sy <package>` **!**  
The way around is to download [pacman-static](https://aur.archlinux.org/pacman-static.git) from **AUR** [built version](https://pkgbuild.com/~eschwartz/repo/x86_64-extracted/pacman-static )

## installation log
`grep -i installed /var/log/pacman.log` recently installed packages
`grep -i upgraded /var/log/pacman.log` recently upgraded packages

## downgrade package
`pacman -U /var/cache/pacman/pkg/<package>`
add to `/etc/pacman.conf`
`IgnorePkg = <package>`
to prevent further updates

## save to list
`pacman -Qqen > <packages>`

## install from list
`pacman -S - < <packages>`

## upgrade
`pacman -Syu`
 
## delete orphans
* `pacman -Rusn $(pacman -Qtdq)` use `-Qttdq` for potential orphans

## delete cache
use `paccache` package.  
* `paccache -rk 1` for keeping only 1 package version (default 3)

## AUR packages
requires `base-devel`, `git`  
```
git clone https://aur.archlinux.org/<package name>.git
cd <package name>
makepkg -fsri
```
