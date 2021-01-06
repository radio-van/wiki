# Contents

- [getty](#getty)
- [tips&trics](#tipstrics)
    - [restore root password](#restore-root-password)
        - [Ubuntu + GRUB](#ubuntu-grub)

# getty
`getty` is a program which detects connection to `tty` (virtual console) and
launches another program (`login` by default).

# tips&trics
## restore root password
### Ubuntu + GRUB
* hold `shift` after BIOS screen to enter `GRUB` menu
* press e to edit boot commands
* append `init=/bin/bash` to the end of the line starting with `linux`
* `Ctrl-x` to boot
* `mount -rw -o remount /` to get read-write access
* `passwd root`
