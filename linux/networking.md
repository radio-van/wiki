# Contents

- [interfaces](#interfaces)
    - [TUN/TAP](#tuntap)
    - [tunneling](#tunneling)
    - [bridge](#bridge)
    - [veth pair](#veth-pair)
    - [Predictable interfaces names](#predictable-interfaces-names)
- [configuration](#configuration)
    - [hostname](#hostname)
    - [interfaces](#interfaces-2)
        - [system V](#system-v)
        - [systemd](#systemd)
        - [netctl](#netctl)
    - [routes](#routes)
    - [firewall](#firewall)
    - [DNS resolving](#dns-resolving)
    - [DHCP server](#dhcp-server)
    - [TFTP](#tftp)
        - [client](#client)
    - [ip address aliasing](#ip-address-aliasing)
    - [wireless](#wireless)
        - [AP](#ap)
        - [wpa supplicant](#wpa-supplicant)
- [hardware](#hardware)
    - [Huawei E3372h](#huawei-e3372h)
    - [Ubiquiti](#ubiquiti)
        - [dd](#dd)
        - [LED patterns](#led-patterns)
        - [recover with TFTP](#recover-with-tftp)
- [tips & trics](#tips-trics)
    - [show process which is listening target port](#show-process-which-is-listening-target-port)
    - [get ip](#get-ip)
    - [capture trafic on particular port](#capture-trafic-on-particular-port)
    - [turn on forwarding](#turn-on-forwarding)
    - [turn off ipv6](#turn-off-ipv6)
    - [dumb Wi-Fi access point](#dumb-wi-fi-access-point)
    - [measure speed](#measure-speed)
    - [flush routing cache](#flush-routing-cache)
    - [which route is used](#which-route-is-used)
    - [priority of route](#priority-of-route)
    - [QR code for WIFI](#qr-code-for-wifi)

# interfaces

## TUN/TAP
overview:: both of them are virtual network kernel interfaces

* **TUN** devices work at the _IP level_ or _layer 3_ level of the network stack and are usually point-to-point connections. A typical use for a TUN device is establishing VPN connections since it gives the VPN software a chance to encrypt the data before it gets put on the wire. Since a *TUN* device works at _layer 3_ it can only accept IP packets and in some cases only IPv4. Additionally because TUN devices work at layer 3 they can’t be used in bridges and don’t typically support broadcasting.
* **TAP** devices work at the _Ethernet level_ or _layer 2_ and therefore behave like a real network adaptor. Since they are running at _layer 2_ they can transport any _layer 3_ protocol and aren’t limited to point-to-point connections. *TAP* devices can be part of a bridge and are commonly used in virtualization systems to provide virtual network adaptors to multiple guest machines. Since *TAP* devices work at _layer 2_ they will forward broadcast traffic which normally makes them a poor choice for VPN connections as the VPN link is typically much narrower than a LAN network (and usually more expensive). But VPN clients using *TAP* interfaces will be in the same subnet and therefor no additional routing is needed (VPN acts as ethernet switch).

## tunneling
**Tunnel** works by using a data portion of a packet (the _payload_) to carry packets that actually provide the service.  
E.g. _SSH tunnel_ picks _TCP_ packets as a payload of own _SSH_ packets and deliver them over _SSH_ connection.  
Another example is sending _IPv6_ packets over _IPv4_.  

## bridge
**Bridge** acts as virtual switch + STP protocol and some other stuff

## veth pair
**VETH** is like physical patch-cord: what comes in on one end, comes out on another

## Predictable interfaces names
* onboard devices (BIOS/Firmware) `enoX`
* PCI Express (BIOS/Firmware) `ensX`
* connected hardware (physical/geographical location) `enpXsY`
* MAC address `enxNNNNNNNNNNNN`
* unpredictable naming `ethX`

# configuration
## hostname
* `echo <hostname> > /etc/hostname`
* `hostname -F /etc/hostname`

for static configurations add to `/etc/hosts`   

* `<static ip>   hostname.domain.com`

## interfaces
### system V
`/etc/networking/interfaces`
```bash
   auto <interface>
   iface <interface> inet loopback|dhcp|manual|static
   ***
```
_***_ additional rules are allowed e.g. `up ip route add default via <address>`   

in case of `static`
```bash
   iface <interface> inet static
       address <address>
       netmask <netmask>
       gateway <gateway>
```
### systemd
use `systemd` units
`/etc/systemd/network/20-<name>.network`
```bash
    [Match]
    Name=<interface>

    [Network]
    DHCP=ipv4
    <or>
    Address=<addresss>/<mask>
    Gateway=<gateway>
```
enable & start unit with `systemctl enable|start 20-<name>`
   
### netctl
`/etc/netctl/...`  
```
Description='...'
Interface=<interface>
Connection=wireless
Security=wpa-configsection
IP=dhcp
WPAConfigSection=(
    'ssid="..."'
    'psk="..."'
    'priority=<N>'
)
```
* higher `<N>` number in priority means that this network will be connected if several networks are present
* `psk` can be obfuscated with `wpa_passphrase <SSID>` (sometimes it causes connection troubles)

Automatic network switching can be enabled by enabling:  
* `netctl-ifplugd@<interface>.service` service for ethernet (requires `ifplugd`)
* `netctl-auto@<interface>.service` service for wireless
* if `WPAConfigSection` is used, most `netctl` options in profile config are ignored, see `WPASupplicant` for proper options  
  for **hidden** network it's better to use `Security=wpa` with `Hidden=yes, Key=... and ESSID=...` options

Automatic networks can be listed with `netctl-auto list` and manually switched with `netctl-auto switch-to <profile>`


## routes
* add gateway `ip route add <gateway IP> dev <interface>`
* add route `ip route add <IP>/<mask> via <gateway> dev <interface>`

## firewall

```
iptables -A INPUT -p tcp -m tcp -m multiport --dports 80,443 -j ACCEPT
...
iptables -A INPUT -m conntrack -j ACCEPT  --ctstate RELATED,ESTABLISHED
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -j DROP
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -j DROP
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -j DROP
```

## DNS resolving
`/etc/resolv.conf`
`nameserver X.X.X.X`

## DHCP server
* assign static address to interface
* install `dhcp`
* config:
```
option domain-name-servers 9.9.9.9;
option subnet-mask 255.255.255.0;
option routers 172.16.0.1;
subnet 172.16.0.0 netmask 255.255.255.0 {
  range 172.16.0.2 172.16.0.250;
}
```
* `systemctl enable --now dhcpd4` (optionally `dhcpd6`)

## TFTP

### client
`curl -T <firmware.bin> tftp://<ip>`


## ip address aliasing
ArchLinux   
* `ip addr add 10.200.10.1/24 dev lo label lo:0`
  permanent:
  `/etc/systemd/network/<NN>-lo-alias.network`
  ```
  [Match]
  Name=lo  
  
  [Address]
  Address=127.0.0.1/8
  [Address]
  Address=::1/128
  [Address]
  Address=10.200.10.1/24
  
  [Route]
  Gateway=127.0.0.1
  Destination=
  [Route]
  Gateway=::1
  Destination=
  [Route]
  Gateway=10.200.10.1
  Destination=
  ```
CentOS
* `/etc/sysconfig/network-scripts`
* `cp ifcfg-ethX icfg-ethX:N`
  
Necessary fields in alias interface are:   
```bash
DEVICE="ethX:N"
BOOTPROTO=static
ONBOOT=yes
IPADDR=...
NETMASK=...
GATEWAY=...
```
* `/etc/init.d/network restart`

## wireless
### AP
* install `hostapd`
* config:
```
ssid=<SSID>
interface=<interface>
driver=nl80211
hw_mode=g
channel=1
auth_algs=1 #WPA ?
wpa=2
wpa_passphrase=<PASSPHRASE>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
* enable `hostapd` service

### wpa supplicant
* create initial config
`/etc/wpa_supplicant/wpa_supplicant.conf`
```bash
    ctrl_interface=/run/wpa_supplicant
    update_config=1
```
* start `wpa_supplicant`
`wpa_supplicant -B -i <interface> -c /etc/wpa_supplicant/wpa_supplicant.conf`
* run `wpa_cli -i <interface>`
    * `scan`
    * `scan_results`
    * `add_network`
    * `set_network <id> ssid "<SSID>"`
    * `set_network <id> psk "<passphrase>"`
    * `enable_network <id>`
    * `save_config`
    * `quit`
* enable `wpa_supplicant.service`

for encrypted passphrase `wpa_passphrase <SSID> <passphrase>` can be used to obtain configuration


# hardware

## Huawei E3372h
```
opkg install kmod-usb-core kmod-usb-net kmod-usb-uhci chat ppp kmod-usb-serial kmod-usb2 kmod-usb3 libusb-1.0 usb-modeswitch kmod-usb-net-rndis kmod-usb-net-cdc-ether kmod-usb-net-huawei-cdc-ncm
```
add new WAN for `eth2`


## Ubiquiti

### dd
* (optional) hard reset by holding **reset** button for 30 sec
* `ip addr add 192.168.1.0/24 dev <iface>`
* `ip route add 192.168.1.20 dev <iface>`
* `scp -O <firmware>.bin ubnt@192.168.1.20:/tmp/`, password is `ubnt`
* **ssh** into and `echo "5edfacbf" > /proc/ubnthal/.uf` (unlock mtd partition)
* check **mtd** with `cat /proc/mtd`, output should look like this:
    ```
    dev:    size   erasesize  name
    mtd0: 00060000 00010000 "u-boot"
    mtd1: 00010000 00010000 "u-boot-env"
    mtd2: 00790000 00010000 "kernel0"
    mtd3: 00790000 00010000 "kernel1"
    mtd4: 00020000 00010000 "bs"
    mtd5: 00040000 00010000 "cfg"
    mtd6: 00010000 00010000 "EEPROM"
    ```
* write OpenWrt into **kernel0** (`mtd2` in example above): `dd if=/tmp/sysupgrade.bin of=/dev/mtdblock2`
* write OpenWrt into **kernel1** (`mtd3` in example above): `dd if=/tmp/sysupgrade.bin of=/dev/mtdblock3`
* boot from **kernel0**: `dd if=/dev/zero bs=1 count=1 of=/dev/mtdblock4`
* reboot

### LED patterns
* _booting_: flashing white every `0.5` sec
* _adoption_: steady white
* _bluetooth_: slow blue
* _operational_: steady blue
* _error_: strobbing white
* _updating firmware_: white-blue
* _tftp_: white-blue-off 5 sec

### recover with TFTP
* `ip addr add 192.168.1.0/24 dev <iface>`
* hold **reset** button, plug power, wait blue-white-off pattern
* `curl -T <official firmware>.bin tftp://192.168.1.20`
* wait for flashing (blue-white pattern -> white)


# tips & trics
## show process which is listening target port
* `ss -ltnp | grep -w ':PORT'`
## get ip
`ip -4 addr show <interface> | grep -oP '(?<=inet\s)\d+(\.\d+){3}`

## capture trafic on particular port
`ngrep port 53`

## turn on forwarding
`sysctl -w net.ipv4.ip_forward=1`

## turn off ipv6
`sysctl net.ipv6.conf.default.disable_ipv6=1`
`sysctl net.ipv6.conf.all.disable_ipv6=1`

## dumb Wi-Fi access point
on Openwrt  
```
    1. Disconnect the (soon-to-be) Dumb AP from your network, and connect your computer to it with an Ethernet cable.
    2. Use the web interface to go to Network → Interfaces and select the LAN interface.
    3. Enter an IP address “next to” your main router on the field “IPv4 address”. (If your main router has IP 192.168.1.1, enter 192.168.1.2). Set DNS and gateway to point into your main router to enable internet access for the dumb AP itself
    4. Then scroll down and select the checkbox “Ignore interface: Disable DHCP for this interface.”
    5. Click “IPv6 Settings” tab and set everything to “disabled”.
    6. In the top menu go to System → Startup, and disable firewall, dnsmasq and odhcpd in the list of startup scripts.
    7. Click the Save and Apply button. Hard-Restart your router if you're not able to connect anymore.
    8. Go to http://192.168.1.2 (or whatever address you specified) and check if the settings for the LAN interface are the same.
    9. Use an Ethernet to connect one of the LAN ports on your main router to one of the LAN/switch ports of your “new” dumb AP. (There's no need to connect the WAN port of the Dumb AP.)
    10. You are done.
```

## measure speed
* server `iperf -B <interface ip> -s -f m`
* client `iperf -c <ip> -f m`

## flush routing cache
`ip r flush cache`

## which route is used
`ip r get <address>`

## priority of route
`ip route add ... metric N` where route with lower `N` is prioritized.  
Also, consider _lookup tables_, e.g. adding _rule_:
`ip rule add to <subnet/mask> priority N lookup <table>`  
will force to lookup specific table first, if `N` is lower than others, can be checked with `ip rule list`


## QR code for WIFI
`WIFI:T:WPA;S:<ssid>;P:<pass>;H:;;`
