# Contents

- [basics](#basics)
- [conditions](#conditions)
- [options](#options)
- [commands](#commands)

# basics
`sed -> awk -> perl`
`awk <condition><command> <file>`

# conditions
could be `/<pattern>/` or `<condition>`
e.g. `awk '$1=="foo"'` will match lines with `foo` in `1`st column

# options
* `END` special marker of the end of the input
* `-F<delimeter>` sets columns delimeter 

# commands
* `{print}` prints matches
* `{print $N}` prints `N`th column of matched line
* `{exit}` exit `awk`, e.g. `awk '{print}NR==10{exit}'` will print first `10` lines

