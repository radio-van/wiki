# Contents

- [init](#init)
- [conflicts on sysupgrade](#conflicts-on-sysupgrade)
- [installation log](#installation-log)
- [downgrade package](#downgrade-package)
- [save to list](#save-to-list)
- [install from list](#install-from-list)
- [upgrade](#upgrade)
- [delete orphans](#delete-orphans)
- [AUR packages](#aur-packages)

# init
* initialize keys `pacman-key --init`
* populate keys `pacman-key --populate <distro>`, `distro` can be found at `/usr/share/pacman/keyrings/`

# conflicts on sysupgrade
```
  error: failed to commit transaction (conflicting files)
  <package> <filepath> exists in filesystem`
```

* check which package owns the file `pacman -Qo <filepath>`
  if it is another package, then it is a bug
* force overwrite file `pacman -S <package> --overwrite <filepath>`

# installation log
`grep -i installed /var/log/pacman.log` recently installed packages
`grep -i upgraded /var/log/pacman.log` recently upgraded packages

# downgrade package
`pacman -U /var/cache/pacman/pkg/<package>`
add to `/etc/pacman.conf`
`IgnorePkg = <package>`
to prevent further updates

# save to list
`pacman -Qqen > <packages>`

# install from list
`pacman -S - < <packages>`

# upgrade
`pacman -Syu`
 
# delete orphans
* `pacman -Rusn $(pacman -Qtdq)` use `-Qttdq` for potential orphans

# AUR packages
requires `base-devel`, `git`  
```
git clone https://aur.archlinux.org/<package name>.git
cd <package name>
makepkg -fsri
```
