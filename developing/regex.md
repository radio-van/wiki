# Contents

- [regex types](#regex types)
- [special character](#special character)
- [groups](#groups)
    - [non capturing group](#groups#non capturing group)
- [lookahead / lookbehind](#lookahead / lookbehind)

# regex types
- **PCRE** `(foo|bar)` means `foo` or `bar`
- **Vim** `\(foo\|\bar\)` otherwise `(`, `)` and `|` will be interpreted as regular characters


# special character
* `\` escaping
* `^` beginning of the line
* `$` end of the line
* `*` any number (or zero) of previous character
* `+` one or more of previous characters [`\+` in plain **grep** or **vim**]
* `?` previous character is optional
* `\S` any non-whitespace character
* `\s` any whitespace character
* `( )` captive grouping, group can be used later as `$<group number>` [`\( \)` and `\<group number` in **vim**]
* `(? )` non-captive grouping, see [lookahead/lookbehind](#lookahead-/-lookbehind)
* `[ ]` characters range, `[a-zA-Z0-9]`
* `[:alnum:]` only letters and numbers (not including russian)


# groups

## non capturing group
Won't be captured as a match.  
`(?:...)`


# lookahead / lookbehind
**Positive lookahead/behind** searches for some expression,
that _is_ followed/preceded by expression in lookahead/behind

**Negative lookahead/behind** searches for some expression,
that _is not_ followed/preceded by expression in lookahead/behind

**PCRE** uses form `(?...)`
* Positive lookahead `(?=...)`
* Negative lookahead `(?!...)`
* Positive lookbehind `(?<=...)`
* Negative lookbehind `(?<!...)`

**Vim** uses form `\@`
* Positive lookahead `\@=`
* Negative lookahead `\@!`
* Positive lookbehind `\@<=`
* Negative lookbehind `\@<!`

e.g. (in **vim** form):
`/foo\(bar\)\@=/` will search for `foo` following `bar` but won't include `bar` in result
