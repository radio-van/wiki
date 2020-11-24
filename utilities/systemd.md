# Contents

- [configuration](#configuration)
- [maintain](#maintain)
    - [reset failed service](#reset-failed-service)

# configuration
if option can be executed several times (e.g. `ExecStart`) it probably should be cleared,  
to prevent unexpected behaviour:  
```bash
ExecStart=
ExecStart=<command>
```
if argument prefixed with `-` then if command/file doesn't exist, it won't raise any errors.  
```bash
ExecStart=-<command>
```
otherwise, if command fails, following comands are not executed  

# maintain
## reset failed service
if a service failed with `start-limit-hit`
it can be restarted with `systemctl reset-failed <unit>`
