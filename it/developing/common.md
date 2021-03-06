# Contents

- [race condition](#race-condition)

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
