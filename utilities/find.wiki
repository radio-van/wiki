= Contents =
    - [[#empty folders|empty folders]]
    - [[#delete|delete]]
    - [[#exec command|exec command]]
    - [[#exec complex commands|exec complex commands]]
    - [[#exec multiple commands|exec multiple commands]]
    - [[#negative condition|negative condition]]
    - [[#operations|operations]]
    - [[#prune|prune]]
    - [[#search pattern|search pattern]]
    - [[#tips&trics|tips&trics]]
        - [[#tips&trics#batch file ranaming|batch file ranaming]]

= empty folders =
`find . -type d -empty`

= delete =
`find . --delete`

= exec command =
`find . -exec foo {} \;`

= exec complex commands =
`find . -exec bash -c ' foo "$1"' _ {} \;`

= exec multiple commands =
`find . -exec foo {} \; -exec bar {} \;`

= negative condition =
use `!`
`find . ! -name 'foo.bar'`

= operations =
* *OR* `-o`
  `expr1 -o expr2` `expr2` is not evaluated if `expr1` is true
  
= prune =
if file is a directory, do not descend into it.
can't be used with `-delete` because it implies `-depth`
e.g.
`find . -path <pattern> -prune -o -print` will exclude folder `<pattern>`
to exclude `<pattern>` in root dir and all subdirs use:
`find . -path "./<pattern>" -prune`

= search pattern =
`find . -path <pattern>` filename matches pattern
pattern is relative to start point directory

= tips&trics =
== batch file ranaming ==
works with recursive paths
`find . -type d -name '<name>' -print0 | xargs --null -I{} sh -c 'echo {} | sed s/<name>/<new-name>/g | xargs -I% mv {} %'`
