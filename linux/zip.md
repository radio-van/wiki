# Contents

    - [Lempel-Ziv compression](#lempel-ziv-compression)
    - [Haffman algorithm](#haffman-algorithm)

Uses an idea of dictionary for repeating words: a word can be put into a dictionary and all its occurs in text can be replaces with link.
For eliminating separate dictionary, links can lead to the positions of words n the same text.
`(pointer, length, [next_symbol])`

## Haffman algorithm
Encoding of each symbol as a fix-length code (like in ASCII) is convinient, but space-consuming.
Instead, principle of Morse code can be used: more rare symbols are encoded with longer code.
Plus special coding can be used, where shorter code do not correlate with longer (like in USA there are no numbers starting with 911).
*Haffman algorithm* searches for the most rare combination of `X` and `Y` symbols and replaces them with combined symbol.
The rate of such combined symbol equals to the sum of rates of `X` and `Y`.

Example:
| symbol | rate |
| A      | 6    |
| B      | 4    |
| C      | 2    |
| D      | 3    |

`[A/6] [B/4] [C/2] [D/3]` - `C` and `D` are rarest, they will be combined with total rate of `2+3=5`
`[A/6] [B/4] [5]` - now `B` and combined-symbol `5` are the rarest, they will be combined
`[A/6] [9]` - on the last iteration, `A` and `9` are combined
`[15]`

Result: Haffman tree
```
   [15]
  /    \
 /      \
[A/6]    [9]
        /   \
   [B/4]    [5]
           /   \
      [C/2]     [D/3]
```

To represent shifts special numbers can be used: `0` to shift left and `1` to shift right:
| symbol | code |
| A      | 0    |
| B      | 10   |
| C      | 110  |
| D      | 111  |

Two key ideas are present: the most common symbols are encoded with shorter code, longer codes are not begun with shorter ones.

