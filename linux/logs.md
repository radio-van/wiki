## Contents =
    - [[#basics|basics]]

## basics =
{{{

| ------------------- kernel space ------------------------ | ----------------------- user space -------------------------- |

                                       +---> /proc/kmsg -----> klogd daemon ---> syslogd daemon ---> output to files in /log
                                       |          |               ^
kernel ---> [kernel ring buffer] ------+          +---------------|------------> systemd-journald
                                       |                          | 
                                       +---> (sys_syslog call) ---+
}}}
