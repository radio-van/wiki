# Contents

- [empty folders](#empty folders)
- [delete](#delete)
- [exec command](#exec command)
- [exec complex commands](#exec complex commands)
- [exec multiple commands](#exec multiple commands)
- [exclude folder](#exclude folder)
- [negative condition](#negative condition)
- [operations](#operations)
- [prune](#prune)
- [search pattern](#search pattern)
- [examples](#examples)
    - [batch file ranaming](#examples#batch file ranaming)
    - [install python requirements](#examples#install python requirements)
    - [search by date](#examples#search by date)
    - [output multiple files in one line](#examples#output multiple files in one line)
    - [loop through list of filenames with spaces](#examples#loop through list of filenames with spaces)

# empty folders
`find . -type d -empty`
# delete
`find . --delete`

# exec command
`find . -exec foo {} \;`

# exec complex commands
`find . -exec bash -c ' foo "$1"' _ {} \;`

# exec multiple commands
`find . -exec foo {} \; -exec bar {} \;`

# exclude folder
`find -name "<pattern>" -not -path "./directory/*"`

# negative condition
use `!`
`find . ! -name 'foo.bar'`

# operations
**OR** `-o`
* `expr1 -o expr2` `expr2` is not evaluated if `expr1` is true
  
# prune
if file is a directory, do not descend into it.
can't be used with `-delete` because it implies `-depth`
e.g.
* `find . -path <pattern> -prune -o -print` will exclude folder `<pattern>`
to exclude `<pattern>` in root dir and all subdirs use:
* `find . -path "./<pattern>" -prune`

# search pattern
* `find . -path <pattern>` filename matches pattern
pattern is relative to start point directory

# examples
## batch file ranaming
works with recursive paths
* `find . -type f -name '<name>' -print0 | xargs --null -I{} sh -c 'echo {} | sed s/<name>/<new-name>/g | xargs -I% mv "{}" "%"'`

## install python requirements
`find ./ -name requirements.txt -exec pip install -r '{}' \;`

## search by date
`find ./ -newermt 2019-01-01 ! -newermt 2020-01-01`

## output multiple files in one line
`find <path> <criteria> -printf '%p '`  
e.g.  
add multiple books in html to *Calibre* (it accepts book in format `calibre <path-to-bbok1> <path-to-book2> ...`)  
`find <path> -type f -name 'index.html' -printf '%p ' | xargs calibre`

## loop through list of filenames with spaces
```sh
find . -type f -name '<pattern>' -exec sh -c '
  for file do
    ...
  done
' exec-sh {} +
```
