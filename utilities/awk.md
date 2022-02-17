# Contents

- [basics](#basics)
- [patterns](#patterns)
- [options](#options)
- [commands](#commands)
- [variables](#variables)
- [processing](#processing)
- [arrays](#arrays)
- [loop](#loop)
- [conditions](#conditions)
- [sorting](#sorting)

# basics
`sed -> awk -> perl`
`awk <condition><command> <file>`

# patterns
could be `/<pattern>/` or `<condition>`
e.g. `awk '$1=="foo"'` will match lines with `foo` in `1`st column

* `awk '/hello/{ print "<pattern match>", $0}'`
* `awk '$4~/hello/{ print "<pattern match in particular field>", $4}'`
* `awk '$4 == "hello"{ print "<exact match>", $4}'`

# options
* `-F<delimiter>` sets columns delimiter 

# commands
* `{print}` prints matches
* `{print $N}` prints `N`th column of matched line
* `{exit}` exit `awk`, e.g. `awk '{print}NR==10{exit}'` will print first `10` lines
* `{printf}` supports C-style formatting:
    e.g. `awk '{ printf "%s \t %-5s", $1, substr($2,1,5)}'`, where:
    * `%-Ns` - right padding for `N` chars
    * `substr($<column>, <from>, <to>)` - shorten line

# variables
* `NF` - last column
* `NR` - number of records
* `FS` - field separator
* `BEGIN`, `END` markers of input?

# processing
```
awk '
BEGIN { print "Start calculating average..." }
      { total = total + $<column> }
END   { print "Result: ", total/NR }
' <file>
```

# arrays
`awk '{ array[<index>]=$<column> ... }'`

# loop
```
awk '
    { for (i in var) {
        print $<column>
        print i
        print array[i]
      }
    }
```
loop over array will loop over its keys:
```
awk '
BEGIN
{ arr["key1"] = "one" }
{ arr["key2"] = "two" }
{ arr["key3"] = "three" }
END

for (i in arr){
    print $i, arr[i]
    }
}'

$:
key1 one
key2 two
key3 three
```

# conditions
```
awk '{
    if (var - 5 > 10)
        print " "
    else if (...)
        ...
}'
```

# sorting
`PROCINFO["sorted_in"]`  # TODO
