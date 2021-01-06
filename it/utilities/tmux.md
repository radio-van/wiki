# Contents

- [configuration](#configuration)
    - [plug-ins](#plug-ins)
        - [tmux-gitbar](#tmux-gitbar)
- [usage](#usage)
    - [panes](#panes)
    - [scripting](#scripting)
        - [basics](#basics)
        - [windows & panes](#windows-panes)
        - [conditions](#conditions)
        - [send keys](#send-keys)
        - [example](#example)

# configuration
## plug-ins
### tmux-gitbar
`branch - remote - ↑[sync to remote] | ●[staged files] x[merge conflict] ⚑[stash file] ✖︎+[changed, but not staged files]...[untracked files]`

# usage
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
