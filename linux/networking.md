# Contents

- [interfaces](#interfaces)
    - [TUN/TAP](#interfaces#TUN/TAP)
    - [bridge](#interfaces#bridge)
    - [veth pair](#interfaces#veth pair)
    - [Predictable interfaces names](#interfaces#Predictable interfaces names)
- [configuration](#configuration)
    - [hostname](#configuration#hostname)
    - [interfaces](#configuration#interfaces)
        - [system V](#configuration#interfaces#system V)
        - [systemd](#configuration#interfaces#systemd)
    - [firewall](#configuration#firewall)
    - [DNS resolving](#configuration#DNS resolving)
    - [DHCP server](#configuration#DHCP server)
    - [ip address aliasing](#configuration#ip address aliasing)
    - [wireless](#configuration#wireless)
        - [AP](#configuration#wireless#AP)
        - [wpa supplicant](#configuration#wireless#wpa supplicant)
- [tips & trics](#tips & trics)
    - [show process which is listening target port](#tips & trics#show process which is listening target port)
    - [get ip](#tips & trics#get ip)
    - [capture trafic on particular port](#tips & trics#capture trafic on particular port)
    - [turn off ipv6](#tips & trics#turn off ipv6)

# interfaces

## TUN/TAP
overview:: both of them are virtual network kernel interfaces

* **TUN** devices work at the _IP level_ or _layer 3_ level of the network stack and are usually point-to-point connections. A typical use for a TUN device is establishing VPN connections since it gives the VPN software a chance to encrypt the data before it gets put on the wire. Since a *TUN* device works at _layer 3_ it can only accept IP packets and in some cases only IPv4. Additionally because TUN devices work at layer 3 they can’t be used in bridges and don’t typically support broadcasting.
* **TAP** devices work at the _Ethernet level_ or _layer 2_ and therefore behave like a real network adaptor. Since they are running at _layer 2_ they can transport any _layer 3_ protocol and aren’t limited to point-to-point connections. *TAP* devices can be part of a bridge and are commonly used in virtualization systems to provide virtual network adaptors to multiple guest machines. Since *TAP* devices work at _layer 2_ they will forward broadcast traffic which normally makes them a poor choice for VPN connections as the VPN link is typically much narrower than a LAN network (and usually more expensive). But VPN clients using *TAP* interfaces will be in the same subnet and therefor no additional routing is needed (VPN acts as ethernet switch).

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

# tips & trics
## show process which is listening target port
* `ss -ltnp | grep -w ':PORT'`
## get ip
`ip -4 addr show <interface> | grep -oP '(?<=inet\s)\d+(\.\d+){3}`

## capture trafic on particular port
`ngrep port 53`

## turn off ipv6
`sysctl net.ipv6.conf.default.disable_ipv6=1`
`sysctl net.ipv6.conf.all.disable_ipv6=1`
