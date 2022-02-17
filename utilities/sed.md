# Contents

- [basics](#basics)
- [arguments](#arguments)
- [commands](#commands)
    - [delete](#commands#delete)
    - [print](#commands#print)
    - [quit](#commands#quit)
    - [substitute](#commands#substitute)
        - [flags](#commands#substitute#flags)
- [patterns](#patterns)
- [ranges](#ranges)
- [recepies](#recepies)
    - [delete everything except alpha-numerical chars](#recepies#delete everything except alpha-numerical chars)
    - [delete color escape codes](#recepies#delete color escape codes)

# basics
`sed -> awk -> perl`
`sed` puts lines from input to *pattern space*, applies commands to them and then prints pattern space to output.
e.g.
`sed ''` will read all lines from file, do nothing (because commands are `''`) to them and print them to output.

`sed "command1; command2; ..." file`

# arguments
* `-n` overrutes default behaviour of printing *pattern space* to output at the end.
* `-i` modify source file (also `-i.orig` will backup source file)
* `-e` combines commands, e.g. `sed -e <command1> -e <command2> ...`

# commands
`/<pattern>/ <action>`

commands can be combined using multiple `-e` argument, or with `;`, e.g. `sed -e 'p' -e 'd'` = `sed 'p; d'`

to apply multiple commands to the same [[#ranges|range]], `<range> {command1, command2}` can be used.

## delete
`/pattern/d` deletes line

## print
`p` prints line (even if `-n` given)  
e.g. `sed -n '2p'` prints 2nd line

## quit
`q` quit (e.g. `sed 10q <file>` will output first `10` lines)

## substitute
`s/pattern/value/[<flag>]` substitutes `pattern` with `value`, `flag` is optional

### flags
* `g` substitute all of multiple occasions of `pattern` instead of just the first.
* `p` prints substituded lines (useful with `-n`)

# patterns
* `\s` any space character
* `^$` blank line
* 
# ranges
* `start,end<command>` apply commands to lines from `start` to `end`
* `/pattern1/,/pattern2/<command>` apply commands from first occasion of `pattern1` to last occasion of `pattern2`
* `$` marks the end of the file

# recepies

## delete everything except alpha-numerical chars
`sed 's/[^[:alnum:]]\+//g'`  
*NOTE:* cyrillic chars are included

## delete color escape codes
`sed -r 's/\x1B\[([0-9]{1,3}(;[0-9]{1,3})*)+m//g'`
