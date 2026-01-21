# Contents

- [Pacman](#pacman)
    - [init](#init)
    - [conflicts on sysupgrade](#conflicts-on-sysupgrade)
    - [mirrorlist](#mirrorlist)
        - [get shortlist by country](#get-shortlist-by-country)
    - [corrupted packages](#corrupted-packages)
    - [partial upgrade](#partial-upgrade)
    - [installation log](#installation-log)
    - [downgrade package](#downgrade-package)
    - [save to list](#save-to-list)
    - [install from list](#install-from-list)
    - [upgrade](#upgrade)
    - [delete orphans](#delete-orphans)
    - [delete cache](#delete-cache)
    - [use aria2 for download](#use-aria2-for-download)
    - [AUR packages](#aur-packages)
- [Flatpak](#flatpak)
    - [cleanup](#cleanup)
    - [pathes](#pathes)
    - [permissions](#permissions)
    - [downgrade](#downgrade)
    - [freeze or pin version](#freeze-or-pin-version)
- [Apt](#apt)
    - [proxy for particular repo](#proxy-for-particular-repo)
    - [skip all interaction](#skip-all-interaction)

# Pacman

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


## mirrorlist

### get shortlist by country
`curl -s "https://archlinux.org/mirrorlist/?country=XX&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 5 -`


## corrupted packages
**error** `... is corrupted (... PGP sign).`  
Update keyring: `pacman -Sy archlinux-keyring`

**NOTE**: always follow this by `pacman -Syu` as [partial upgrades](##partial upgrade) is a bad idea

if doesn't help, additional steps are required:
```
rm -R /etc/pacman.d/gnupg/
rm -R /root/.gnupg/ 
gpg --refresh-keys
pacman-key --init && pacman-key --populate archlinux
# pacman-key --refresh-keys  # optional
```

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

## use aria2 for download
`pacman -Sp [package] | sudo aria2c -d /var/cache/pacman/pkg/ -i -`
or in config:
`XferCommand = /usr/bin/aria2c --allow-overwrite=true --continue=true --file-allocation=none --log-level=error --max-tries=2 --max-connection-per-server=2 --max-file-not-found=5 --min-split-size=5M --no-conf --remote-time=true --summary-interval=60 --timeout=5 --dir=/ --out %o %u`

## AUR packages
requires `base-devel`, `git`  
```
git clone https://aur.archlinux.org/<package name>.git
cd <package name>
makepkg -fsri
```


# Flatpak

## cleanup
- `flatpak uninstall --unused`
- `sudo ostree prune --repo=/var/lib/flatpak/repo`

## pathes
`$HOME/.var`

## permissions
`flatpak override <package> --filesystem=<path>`

## downgrade
* `flatpak remote-info --log flathub <package>`
* `flatpak update --commit=<commit> <package>`

## freeze or pin version
* `flatpak mask <package>`
* `flatpak mask --remove <package>`


# Apt

## proxy for particular repo
add to `/etc/apt/apt.conf.d/99proxy`:
```
Acquire::http::Proxy {
    your.local.repository "http://...";
};
```
`DIRECT` can be used instead of `<proxy>` for opposite config


## skip all interaction
`DEBIAN_FRONTEND=noninteractive apt-get install -y <packages>`
