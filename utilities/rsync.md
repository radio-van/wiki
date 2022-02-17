# Contents

- [options](#options)
- [progress](#progress)
- [protocol](#protocol)
    - [ssh](#protocol#ssh)
        - [non-default ssh port](#protocol#ssh#non-default ssh port)

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

# protocol
## ssh
### non-default ssh port
add `-e 'ssh -p <port>'` to **rsync** arrguments
