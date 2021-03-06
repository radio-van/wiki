# Contents

- [configuration](#configuration)
    - [basics](#configuration#basics)
    - [history across all shells](#configuration#history across all shells)
- [conditions](#conditions)
    - [compare strings](#conditions#compare strings)
    - [substring in string](#conditions#substring in string)
- [escape sequences](#escape sequences)
- [pipes](#pipes)
    - [conditions](#pipes#conditions)
    - [group conditions](#pipes#group conditions)
    - [run several commands simultaneously](#pipes#run several commands simultaneously)
- [scripting](#scripting)
    - [set options](#scripting#set options)
    - [redirect output](#scripting#redirect output)
    - [compare two dirs](#scripting#compare two dirs)
    - [get name of current function](#scripting#get name of current function)
    - [get name of current script?](#scripting#get name of current script?)
    - [get args](#scripting#get args)
    - [pass args to func](#scripting#pass args to func)
    - [loops](#scripting#loops)
        - [loop list](#scripting#loops#loop list)
        - [loop words in string](#scripting#loops#loop words in string)
        - [loop lines in file](#scripting#loops#loop lines in file)
        - [loop while command succeeded](#scripting#loops#loop while command succeeded)
    - [concat strings](#scripting#concat strings)
- [variables](#variables)
    - [expansion](#variables#expansion)
- [usage](#usage)
    - [execute and source](#usage#execute and source)
    - [previous command and arguments](#usage#previous command and arguments)
    - [replace symbols in previous command](#usage#replace symbols in previous command)
    - [empty file](#usage#empty file)
    - [check if file is empty](#usage#check if file is empty)
    - [get basename of file](#usage#get basename of file)
    - [print current terminal emulator](#usage#print current terminal emulator)
    - [generate random password](#usage#generate random password)
    - [get output from process executed by another thread](#usage#get output from process executed by another thread)

# configuration
## basics
* **login shell** when you login from another host or text console on login machine
* **non-login shell** when you start display manager
* **interactive shell** when you're connected to a pseudo-terminal via text-console or SSH
* **non-interactive shell** rare, can be launched on some display managers or when shell runs a script

* `.bashrc` - _interactive_ and _non-login_ shell (i.e in existing session)
* `.bash_profile` - _login_ shell

## history across all shells
Basic way
`export PROMPT_COMMAND='history -a'`
      If set, the value is executed as a command prior to issuing each primary prompt.
      So every time a command has finished, it appends the unwritten history item to `~/.bash_history` before displaying the prompt (only `$PS1`) again.
Complex way
`export HISTSIZE=50000`
      Maximum number of history lines in memory
`export HISTFILESIZE=50000`
      Maximum number of history lines on disk
`export HISTCONTROL=ignoredups:erasedups`
      Ignore duplicate lines
`shopt -s histappend`
      When the shell exits, append to the history file instead of overwriting it
`export PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a; history -c; history -r"`
      After each command, append to the history file and reread it

# conditions
## compare strings
```bash
[[ $var == "string" ]] && <equal> || <not equal>
```
## substring in string
```bash
[[ $substring == *"string"* ]] && <included> || <not included>
```

# escape sequences
`\e` and `\033` are equal  

* `\e[1A` go to previous line
* `\e[<N><D>` go `N` lines/columns in direction `<D>`:
    * `A` up
    * `B` down
    * `C` forward
    * `D` backward
* `\e[K` clear line
* `\e[0m` default color
* `\e[0;3<N>m` color `N`
* `\e[1;3<N>m` **bright** color `N`
* `\e[2J` clear the screen
* `\e[s` save cursor position
* `\e[u` restore cursor position

# pipes
## conditions
* successful (i.e. one that returns `0`) command passes the sequence to the first command after `&&`
* unsuccessful command (return != `0`) passes sequence to the first command after `||`
* the next "choice" of following command is based on return signal of the second command

e.g.  
`<com0> && <comYES> || <comNO> && <comNEXT>`
- if `com0` returns `0`, `comYES` and `comNEXT` are executed
- if `com0` return != `0`, `comNO` and `comNEXT` are executed
- if `cmm0` is OK and `comYES` fails, `comNO` and `comNEXT` are executed

## group conditions
`<condition> && ( this && will && run_only_on_success ; ) || ( this_will_run_only && on_fail ; )`

## run several commands simultaneously
`<command1> & <command2>`

# scripting
## set options
* `set -e` will force script to exit after first failed command (but it won't catch fails in the middle of pipe)
```bash
   set -e
   failing_command                # with set -e script exits here
   fail_could_be_ignored || true  # force to ignore command results
   normal_command                 # this won't be executed with set -e
```
* `-o pipefail` sets pipe command exit status to the non-zero status of rightmost command in pipe, i.e. if one of pipe commands fails, the whole pipe fails
* `-u` treat unset variables as errors and exit, instead of empty output
* `-x` print each command before executing (with proper arguments)

## redirect output
* `<command> &> <smth>` redirects everything
* `<command> 2> <smth>` redirects `stderr`

## compare two dirs
* show same files in 2 dirs, ignoring extensions
`comm -12 <(find dir1 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort) <(find dir2 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort)`
* show different files in 2 dirs, ignoring extensions
`comm -3 <(find dir1 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort) <(find dir2 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort)`

## get name of current function
`$FUNCNAME`

## get name of current script?
`$0`

## get args
`$@` - all args
`$1, $2, ...` - first, second, etc arg
`${@:-1}` last arg

## pass args to func
```
inner_func() { <do smth>; }
outer_func() { <do smth>; "$@"; <do smth>; }

outer_func inner_func arg1 arg2
```

## loops
### loop list
```bash
for <loop_var> in <word1> <word2> <word3>; do
  <command>
done
```

### loop words in string
```bash
IFS=' '
for <loop_var> in <string>; do
  <command>
done
```

### loop lines in file
```bash
while read -r <loop_var>
do
  <command>
done < file
```

```bash
while read -r <loop_var>
do
  <command>
done <<< <multiline_variable>
```

### loop while command succeeded
```bash
while ! (command1 && command2 && ...); do echo 'retrying'; done
```

or  

```bash
until $(<command>)
do
  echo "retrying"
done
```

## concat strings
```bash
foo="Hello"
foo="${foo} World"
```
or  
```bash
foo+="<text>"
```

# variables
## expansion
`${VAR:N}` cuts first `N` symbols
`${VAR:N:M}` cuts first `N` symbols and then output `M` symbols of what is rest.  
e.g.  
`${var:0:N}` returns first `N` symbols

`${VAR#<pattern>}` delete `<pattern>` from the beginning (matches the minimal possible pattern)
`${VAR%<pattern>}` delete `<pattern>` from the end (matches the minimal possible pattern)

`${VAR##<pattern>}` delete `<pattern>` from the beginning (matches the maximum possible pattern)
`${VAR%%<pattern>}` delete `<pattern>` from the end (matches the maximum possible pattern)
*NOTE:* globes (`*`) are supported

e.g.
`var="apple orange"`
`echo "${var#ap}` results `ple orange`
`echo "${var#*or}` results `ange`
`echo "${var#*}` results `apple orange` (because of minimal pattern)
`echo "${var##*}` results ` ` (because of maximum pattern)

# usage
## execute and source
* `execute` aka `./<script>` runs script in _new_ shell (all changes to environment will stay in that shell)
* `source` aka `. <script>` runs script in _current_ shell (all changes to environment will be applied to current shell)

## previous command and arguments
- `!!` whole previous command (useful with `sudo`)
- `!^` first argument
- `!$` last argument
- `!*` all arguments
- `!:2` second argument
- `!:2-3` second to third
- `!:2-$`, `!:2*` second to last
- `!:2-` second to last, *does not* includes last
- `!:0` the command itself

## replace symbols in previous command
- `^X^Y` replaces `X` with `Y` and executes last command
- `!!:gs/foo/bar/` replace all `foo` with `bar` and executes previous command

## empty file
`> <filename>`

## check if file is empty
`[ -s <filename> ]`

## get basename of file
`basename <path>`

## print current terminal emulator
`ps -o comm= -p "$(($(ps -o ppid= -p "$(($(ps -o sid= -p "$$")))")))"`

## generate random password
`< /dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-32};echo;`

## get output from process executed by another thread
`strace -p<PID> -s9999 -e write`  
- `write` means capture output
- `-s9999` avoids truncating to 32 chars
