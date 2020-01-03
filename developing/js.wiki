= Contents =
    - [[#logging|logging]]
        - [[#logging#assert|assert]]
        - [[#logging#clear|clear]]
        - [[#logging#css|css]]
        - [[#logging#group|group]]
        - [[#logging#json|json]]
        - [[#logging#levels|levels]]
        - [[#logging#memory|memory]]
        - [[#logging#placeholders|placeholders]]
        - [[#logging#time|time]]
        - [[#logging#trace|trace]]

= logging =
== assert ==
output only if assertion is *False*
`console.assert(<condition>, <text>);`

== clear ==
`console.clear()`

== css ==
`console.log('%c<text>','color: <color>; font-size: <size, e.g. x-large>');`
`%c` should be before the part that is needed to be changed

== group ==
{{{
    console.group();
       console.log(...);
       console.log(...);
    console.groupEnd()
}}} 

== json ==
* output object in json form
`console.dir()`
* output *json* list as table
`console.table()`

== levels ==
`console.log()`
`console.debug()`
`console.error()`

== memory ==
measure memory consumption
`console.memory()`

== placeholders ==
`%o` — object
`%s` — string
`%d` — integer/float

== time ==
measure time of code block execution
{{{
    console.time(<text>)
      ...code...
    console.timeEnd(<text>)
}}}

== trace ==
ouptut call stack
`console.trace()`
