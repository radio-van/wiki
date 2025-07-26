# Contents

- [options](#options)
- [progress](#progress)
- [tips and tricks](#tips-and-tricks)
    - [non-default ssh port](#non-default-ssh-port)
    - [sudo on other side](#sudo-on-other-side)

# options
`-az` default options:  `archive`, `compression`  
`--progress` show progress of copying  
`-n --dry-run` do not copy, just test  
`-I` ignore files in destination (force replace)  
`--ignore-existing` do not copy files with the same name even if they are newer  
`--update` copy files only if they are newer than ones in the destination  
`--delete` delete files in destination  
`-P` combines `--progress` and `--partial`  
`--remove-source-files` remove source after transfer  


# progress
`--info=progress2` without `-v` and `-P` flags shows overall progress


# tips and tricks
## non-default ssh port
add `-e 'ssh -p <port>'` to **rsync** arrguments

## sudo on other side
* `read -s -p "Remote sudo password: " SUDOPASS && rsync -avzuP --stats --rsync-path="echo $SUDOPASS | sudo -Sv && sudo rsync" <host> <target>`
OR  
- add `<username> ALL=NOPASSWD:/usr/bin/rsync` to sudoers on the other side
- use `--rsync-path='sudo rsync'`
