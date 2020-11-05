# Contents

- [logging](#logging)
    - [assert](#assert)
    - [clear](#clear)
    - [css](#css)
    - [formatted strings](#formatted-strings)
    - [group](#group)
    - [json](#json)
    - [levels](#levels)
    - [memory](#memory)
    - [placeholders](#placeholders)
    - [time](#time)
    - [trace](#trace)
- [translations](#translations)

# logging
## assert
output only if assertion is *False*
`console.assert(<condition>, <text>);`

## clear
`console.clear()`

## css
`console.log('%c<text>','color: <color>; font-size: <size, e.g. x-large>');`
`%c` should be before the part that is needed to be changed

## formatted strings
```js
    console.log(`<text> ${<var>}`)
```

## group
```js
    console.group();
       console.log(...);
       console.log(...);
    console.groupEnd()
```

## json
* output object in json form
`console.dir()`
* output *json* list as table
`console.table()`

## levels
`console.log()`
`console.debug()`
`console.error()`

## memory
measure memory consumption
`console.memory()`

## placeholders
`%o` — object
`%s` — string
`%d` — integer/float

## time
measure time of code block execution
```js
    console.time(<text>)
      ...code...
    console.timeEnd(<text>)
```

## trace
ouptut call stack
`console.trace()`

# translations
* create `.pot` file:  
`./node_modules/.bin/gulp extract-strings`
* update `.po` (e.g. `locale/angular/ru.po`) file from `.pot` file in `PoEdit`
* enter new translations
* compile new translations:  
`./node_modules/.bin/gulp compile-translations`
