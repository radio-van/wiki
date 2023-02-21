# Contents

- [Configuration](#configuration)
    - [plug-ins](#plug-ins)
        - [tmux-gitbar](#tmux-gitbar)
- [Usage](#usage)
    - [keybindings](#keybindings)
    - [panes](#panes)
    - [scripting](#scripting)
        - [basics](#basics)
        - [windows & panes](#windows-panes)
        - [conditions](#conditions)
        - [send keys](#send-keys)
        - [example](#example)
- [Troubleshooting](#troubleshooting)
    - [ssh-agent forwarding](#ssh-agent-forwarding)
- [Useful links](#useful-links)

# Configuration

## plug-ins
### tmux-gitbar
`branch - remote - ↑[sync to remote] | ●[staged files] x[merge conflict] ⚑[stash file] ✖︎+[changed, but not staged files]...[untracked files]`


# Usage

## keybindings
`list-keys` prints actual commands binded to keys

## panes

list panes & windows:: `^b s`, `tmux ls`

## scripting
### basics
* **target** `-t` `session:window.pane`, blank means _current_

### windows & panes
* combine windows into one
`join-pane -s <source> -t <target>`
* separate panes into windows
`break-pane`
* count number of panes
`tmux list-panes | wc -l`
* restart command in pane, if `-k` given, kills current command
`: respawn-pane [-k]`

### conditions
`if-shell 'tmux <condition>' 'result is true' 'result is false'`
example: `if-shell 'tmux select-window -t :0' '' 'new-window -t :0'`

### send keys
`tmux send-keys -t "<target>" <shell-command> Enter`

### example
create session, run commands and attach to it
```bash
    tmux new-session -d '<shell-command>'
    tmux split-window -v '<another-shell-command>'
    tmux split-window -h
    tmux new-window '<third-shell-command>'
    tmux -2 attach-session -d
```

# Troubleshooting

## ssh-agent forwarding
If **Tmux** session existed before connection with **ssh-agent** forwarding, 
it lacks the value of `SSH_AUTH_SOCK` env variable.  
It can be exported manually or added to file which is sourced via `.ssh/rc`.
In this case, add
```
Host *
  IdentityAgent <symlinked file>
```
to `.ssh/config` on remote host.

`.ssh/rc` on remote host can be:
```bash
#!/bin/bash
# update SSH_AUTH_SOCK on each connection

if test "$SSH_AUTH_SOCK"; then
  ln -sf $SSH_AUTH_SOCK ~/.ssh/auth_sock
fi
```

# Useful links
* [How to start tmux with several panes open at the same time?](https://askubuntu.com/questions/830484/how-to-start-tmux-with-several-panes-open-at-the-same-time)
* [tmux shortcuts & cheatsheet](https://gist.github.com/MohamedAlaa/2961058)
* [Start up tmux with custom windows, panes and applications running](https://gist.github.com/todgru/6224848)
* [tmux-config: Tmux configuration](https://github.com/samoshkin/tmux-config)
* [tmux-resurrect: Persists tmux environment](https://github.com/tmux-plugins/tmux-resurrect)
* [Getting started with Tmux](https://linuxize.com/post/getting-started-with-tmux/)
* [Bash scripts with tmux to launch a 4-paned window](https://stackoverflow.com/questions/5447278/bash-scripts-with-tmux-to-launch-a-4-paned-window)
* [shell - How to set up tmux so that it starts up with specified windows opened?](https://stackoverflow.com/questions/5609192/how-to-set-up-tmux-so-that-it-starts-up-with-specified-windows-opened)
* [optionally run several commands when starting tmux](https://superuser.com/questions/575909/optionally-run-several-commands-when-starting-tmux)
* [tmux - How to send keys to "other pane"?](https://superuser.com/questions/744857/how-to-send-keys-to-other-pane)
* [Tmux - Execute command in session on startup](https://superuser.com/questions/863538/tmux-execute-command-in-session-on-startup)
