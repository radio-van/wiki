= Contents =
    - [[#configuration|configuration]]
        - [[#configuration#basics|basics]]
        - [[#configuration#history across all shells|history across all shells]]
    - [[#scripting|scripting]]
        - [[#scripting#set options|set options]]
        - [[#scripting#recepies & examples|recepies & examples]]
            - [[#scripting#recepies & examples#compare two dirs|compare two dirs]]
            - [[#scripting#recepies & examples#get name of current function|get name of current function]]
            - [[#scripting#recepies & examples#get name of current script?|get name of current script?]]
            - [[#scripting#recepies & examples#get args|get args]]
            - [[#scripting#recepies & examples#loop words in string|loop words in string]]
            - [[#scripting#recepies & examples#run a command for each line of a file|run a command for each line of a file]]
            - [[#scripting#recepies & examples#wait until command returns true|wait until command returns true]]
    - [[#variables|variables]]
        - [[#variables#variable modification|variable modification]]
    - [[#usage|usage]]
        - [[#usage#execute and source|execute and source]]
        - [[#usage#parameters expansion|parameters expansion]]
        - [[#usage#previous command and arguments|previous command and arguments]]
        - [[#usage#replace symbols in last command|replace symbols in last command]]

= configuration =
== basics ==
login shell:: when you login from another host or text console on login machine
non-login shell:: when you start display manager
interactive shell:: when you're connected to a pseudo-terminal via text-console or SSH
non-interactive shell:: rare, can be launched on some display managers or when shell runs a script

* `.bashrc` - _interactive_ and _non-login_ shell (i.e in existing session)
* `.bash_profile` - _login_ shell

== history across all shells ==
Basic way::
`export PROMPT_COMMAND='history -a'`
::    If set, the value is executed as a command prior to issuing each primary prompt.
      So every time a command has finished, it appends the unwritten history item to `~/.bash_history` before displaying the prompt (only `$PS1`) again.
Complex way::  
`export HISTSIZE=50000`
::    Maximum number of history lines in memory
`export HISTFILESIZE=50000`
::    Maximum number of history lines on disk
`export HISTCONTROL=ignoredups:erasedups`
::    Ignore duplicate lines
`shopt -s histappend`
::    When the shell exits, append to the history file instead of overwriting it
`export PROMPT_COMMAND="${PROMPT_COMMAND:+$PROMPT_COMMAND$'\n'}history -a; history -c; history -r"`
::    After each command, append to the history file and reread it

= scripting =
== set options ==
* `set -e` will force script to exit after first failed command (but it won't catch fails in the middle of pipe)
{{{
   set -e
   failing_command                # with set -e script exits here
   fail_could_be_ignored || true  # force to ignore command results
   normal_command                 # this won't be executed with set -e
}}}
* `-o pipefail` sets pipe command exit status to the non-zero status of rightmost command in pipe, i.e. if one of pipe commands fails, the whole pipe fails
* `-u` treat unset variables as errors and exit, instead of empty output
* `-x` print each command before executing (with proper arguments)

== recepies & examples ==
=== compare two dirs ===
* show same files in 2 dirs, ignoring extensions
`comm -12 <(find dir1 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort) <(find dir2 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort)`
* show different files in 2 dirs, ignoring extensions
`comm -3 <(find dir1 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort) <(find dir2 -type f -exec bash -c 'basename "${0%.*}"' {} \; | sort)`

=== get name of current function ===
`$FUNCNAME`

=== get name of current script? ===
`$0`

=== get args ===
`$@` - all args
`$1, $2, ...` - first, second, etc arg

=== loop words in string ===
{{{
IFS=' '
for <loop_var> in <string>; do
  <command>
done
}}}

=== run a command for each line of a file ===
{{{
while read -r <loop_var>
do
  <command>
done < file
}}}

{{{
while read -r <loop_var>
do
  <command>
done <<< <multiline_variable>
}}}

=== wait until command returns true ===
{{{
until $(<command>)
do
  echo "Try again"
done
}}}

= variables =
== variable modification ==
`${VAR:N}` cuts first `N` symbols
`${VAR:N:M}` cuts first `N` symbols and then output `M` symbols of what is rest.

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

= usage =
== execute and source ==
* `execute` aka `./<script>` runs script in _new_ shell (all changes to environment will stay in that shell)
* `source` aka `. <script>` runs script in _current_ shell (all changes to environment will be applied to current shell)

== parameters expansion ==
| `${VAR#pattern}`  | delete shortest match of pattern from the beginning |
| `${VAR##pattern}` | delete longest match of pattern from the beginning  |
| `${VAR%pattern}`  | delete shortest match of pattern from the end       |
| `${VAR%%pattern}` | delete longest match of pattern from the end        |

== previous command and arguments ==
- `!!` whole previous command (useful with `sudo`)
- `!^` first argument
- `!$` last argument
- `!*` all arguments
- `!:2` second argument
- `!:2-3` second to third
- `!:2-$`, `!:2*` second to last
- `!:2-` second to last, *does not* includes last
- `!:0` the command itself

== replace symbols in last command ==
- `^X^Y` replaces `X` with `Y` and executes last command
- `!!:gs/foo/bar/` replace all `foo` with `bar` and executes previous command
