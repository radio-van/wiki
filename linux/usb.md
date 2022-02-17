# Contents

- [topology](#topology)
- [control power](#control power)
- [plug/unplug](#plug/unplug)

# topology
For each device there are **Bus** and **Device** number.  
*Root* hubs are always `Device 1` on each **Bus**.  

# control power
If hub supports this (e.g. **D-Link DAB-H7**), it can be controlled via `uhubctl`:  
`uhubclt -l <hub num> -a on|off -p <port>`  

`hub num` can be obtained from running `uhubctl` without arguments.

# plug/unplug
`echo <path> > /sys/bus/usb/drivers/usb/unbind`  
where `path` is in form of `bus-port-subport`, e.g. `3-1.5` which means that USB  
hub is connected to port `1` of bus `3` and after applying this command  
device on port `5` of the hub will be disconnected.

To plug device back:  
`echo <path> > /sys/bus/usb/drivers/usb/bind`
