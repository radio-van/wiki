# Contents

- [basics](#basics)
- [journald](#journald)
    - [disk usage](#journald#disk usage)
    - [clean](#journald#clean)

# basics
```
| ------------------- kernel space ------------------------ | ----------------------- user space -------------------------- |

                                       +---> /proc/kmsg -----> klogd daemon ---> syslogd daemon ---> output to files in /log
                                       |          |               ^
kernel ---> [kernel ring buffer] ------+          +---------------|------------> systemd-journald
                                       |                          | 
                                       +---> (sys_syslog call) ---+
```

# journald

## disk usage
* `/var/log/journal`
* `journalctl --disk-usage`

## clean
- `journalctl --rotate` archive current logs
- `journalctl --vacuum-time=N` e.g. `N=1d`
