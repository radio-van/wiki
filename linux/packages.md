## Contents =
    - [[#conflicts on sysupgrade|conflicts on sysupgrade]]
    - [[#installation log|installation log]]
    - [[#downgrade package|downgrade package]]

## conflicts on sysupgrade =
{{{
  error: failed to commit transaction (conflicting files)
  <package> <filepath> exists in filesystem`
}}}

* check which package owns the file `pacman -Qo <filepath>`
  if it is another package, then it is a bug
* force overwrite file `pacman -S <package> --overwrite <filepath>`

## installation log =
`grep -i installed /var/log/pacman.log` recently installed packages
`grep -i upgraded /var/log/pacman.log` recently upgraded packages

## downgrade package =
`pacman -U /var/cache/pacman/pkg/<package>`
add to `/etc/pacman.conf`
`IgnorePkg = <package>`
to prevent further updates
