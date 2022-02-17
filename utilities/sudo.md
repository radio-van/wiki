# Contents

- [basics](#basics)
- [allow sudo command](#allow sudo command)
- [user ban](#user ban)

# basics
Rules for `sudo` command are determined in `sudoers` file.   
This file must be always edited via `visudo` command.   
Pass `EDITOR=<path_to_editor>` if needed.   

# allow sudo command
User must be in `sudo` or `wheel` group.<br>
`usermod -aG <group> <user>` (`-a` for preserve existing list of groups, `-G` for new list of groups)

Uncomment corresponding lines in `sudoers` file to allow members of`sudo`/`wheel` group<br>
to use `sudo` command.

Generic form of line is:
`<users> <hosts>=(<effective_users>) <tag> <commands>`
where:
- `users` are users who can run sudo
- `hosts` are hosts on which they can run sudo
- `effective_users` list of users they must be running as *?*
- `tag` e.g. `NOPASSWD`
- `commands` filter list of commands allowed to run, in form `/path/command1, /path/command2, ...`

# user ban
reset ban: `faillock --user <username> --reset`
