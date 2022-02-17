# Contents

- [configuration](#configuration)
    - [change container storage path](#change-container-storage-path)
    - [virtual bridge](#virtual-bridge)
- [usage](#usage)
    - [attach](#attach)
    - [create](#create)
    - [add bridge to default container config](#add-bridge-to-default-container-config)
    - [lxc clones](#lxc-clones)
    - [fix alpine tty bug](#fix-alpine-tty-bug)
    - [mount](#mount)

# configuration
## change container storage path
`echo 'lxc.lxcpath=...' >> /etc/lxc/lxc.conf`
default path is `/var/lib/lxc/`

## virtual bridge
virtual bridge allows containers to communicate with each other and host
`/etc/default/lxc-net`
```
    USE_LXC_BRIDGE="true"

    LXC_ADDR="10.0.0.1"
    LXC_NETMASK="255.255.255.0"
    LXC_NETWORK="10.0.0.0/24"
```
for DHCP
```
    LXC_DHCP_RANGE="10.0.0.2,10.0.0.254"
    LXC_DHCP_MAX="253"
```
enable service
`systemctl enable lxc-net.service`

# usage
## attach
`lxc-attach <name> -- <command>` can be used to run commands inside container.
`lxc-attach` uses (?) `$PATH` from host, so `/bin` can be not presented in `PATH` and therefore
common utils like `ls`, `cat`, `echo` won't be found. To fix this manually add `/bin` to `$PATH` or
run `lxc-attach` with `--clear-env` option.

## create
`lxc-create -n <name> -t download -- --dist <distributive> --release <dist version> --arch <architecture>`

## add bridge to default container config
`/etc/lxc/default.conf`
```
    lxc.net.0.type = veth
    lxc.net.0.link = lxcbr0
    lxc.net.0.flags = up
```

## lxc clones
`lxc-copy -n base -N clone -B overlayfs -s`

## fix alpine tty bug
add to `/etc/securetty` inside container
```
    lxc/console
    lxc/tty1
    lxc/tty2
    lxc/tty3
    lxc/tty4
```

## mount
e.g. squid requires `/dev/shm`
`lxc.mount.entry = none dev/shm tmpfs nodev,nosuid,noexeci,mode=1777,create=dir 0 0`
