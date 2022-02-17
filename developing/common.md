# Contents

- [race condition](#race condition)
- [task complexion](#task complexion)
    - [Big-O notation](#task complexion#Big-O notation)

# race condition
Happens when program, to operate properly, depends on sequence or timing of the program's processes or threads,
especially when the processes depend on some shared state.
Bug is often hard to reproduce because it depends on relative timing.

Example:
* expected behaviour

| Thread1        | Thread2        | Action   | Value |
| -------------- | -------------- | -------- | ----- |
| read value     |                | <--      | 0     |
| increase value |                |          | 0     |
| write value    |                | -->      | 1     |
|                | read value     | <--      | 1     |
|                | increase value |          | 1     |
|                | write value    | -->      | 2     |

* race condition

| Thread1        | Thread2        | Action | Value |
| -------------- | -------------- | ------ | ----- |
| read value     |                | <--    | 0     |
|                | read value     | <--    | 0     |
| increase value |                |        | 0     |
|                | increase value |        | 0     |
| write value    |                | -->    | 1     |
|                | write value    | -->    | 1     |

# task complexion

## Big-O notation
* `O(1)` - constant time, independent of number of items
* `O(N)` - time is proportional to number of items
* `O(log N)` - time is proportional to `log(N)`

Basically `O` means that an operation will take time up to a maximum  
of `k*f(N)` where:
- `k` is a constant multiplier
- `f()` is a function that depends on `N`
