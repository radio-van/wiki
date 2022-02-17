# Contents

- [configuration](#configuration)
- [maintain](#maintain)
    - [reset failed service](#maintain#reset failed service)
- [restart service on file changed](#restart service on file changed)

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

# restart service on file changed
`/etc/systemd/system/<service>.service`
```
[Unit]
Description=<service>
After=network.target

[Service]
Type=simple
WorkingDirectory=<service path>
ExecStart=<service command>
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/<service>-watcher.service`
```
[Unit]
Description=<restarter>
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl restart <service>.service

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/<service>-watcher.path`
```
[Path]
PathModified=<workdir path>

[Install]
WantedBy=multi-user.target
```
