# Contents

- [configuration](#configuration)
- [add rule](#add-rule)
- [list rules](#list-rules)
- [delete rule](#delete-rule)
- [flush rules](#flush-rules)
- [recepies](#recepies)
    - [open port](#open-port)
    - [port forwarding](#port-forwarding)
    - [trafic forwarding](#trafic-forwarding)
    - [basic routing between eth0 and wlan0](#basic-routing-between-eth0-and-wlan0)
    - [allow trafic between particular WG clients](#allow-trafic-between-particular-wg-clients)

# configuration
* save rules `iptables-save > /etc/iptables/iptables.rules`
* restore rules `iptables-restore < /etc/iptables/iptables.rules`


# add rule
`iptables -t <table> -A <chain> -p <protocol> -j <target>`  
useful options:  
* `--dport <port>` destination port of incoming traffic
* `--to-destination <address>:<port>` destination for filtered traffic
* `--match`, `-m` additional filters
* `-i` aka incoming to specific port
* `-o` aka outcoming from specific port

# list rules
`iptables -n -t <table> -L --line-number`  
* `-n` disables hostname conversion  
* `-v` verbose, shows also interfaces for `MASQUERADE` rules  


# delete rule
list table with `--line-numbers`  
`iptables -D <CHAIN_NAME> <RULE_NUMBER>`  

also rule can be deleted if it is repeated with `-D` instead of `-A` option.


# flush rules
`iptables -t <table> -F` flushes whole table
`iptables -t <table> -F <chain>` flushes single chain


# recepies

## open port
`iptables -A INPUT -p tcp --dport <PORT> -j ACCEPT`

## port forwarding
w/o NAT  
`iptables -t nat -A PREROUTING -i eth0 -p tcp --dport $PORT -j REDIRECT --to-port $PORT`  
w/ NAT  
`iptables -t nat -A PREROUTING -p tcp --dport $PORT -j DNAT --to-destination $ADDRESS:$PORT`  

## trafic forwarding
`iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT` allow forwarding from `eth0` to `eth1`

## basic routing between eth0 and wlan0
```bash
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
   sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
   sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT  
```
`--state RELATED,ESTABLISHED` allows packet for already established connection.  
e.g. if firewall rejects any incoming connections, it will reject response from remote webserver, othervise it is  
allowed as ESTABLISHED connection from allowed outcoming request from the host.


## allow trafic between particular WG clients
```bash
# allow trafffic to outside
iptables -t nat -A POSTROUTING -s {{WG NET}} -o {{WAN}} -j MASQUERADE;
# allow incoming traffic to WG
iptables -A INPUT -p udp -m udp --dport {{WG PORT}} -j ACCEPT;

# allow traffic from and to particular client
iptables -A FORWARD -i wg0 -o wg0 -s <client IP> -j ACCEPT;
iptables -A FORWARD -i wg0 -o wg0 -d <client IP> -j ACCEPT;
# forbit other client-to-client communication
iptables -A FORWARD -i wg0 -o wg0 -j DROP;
# allow traffic to and from WG interface
iptables -A FORWARD -i wg0 -j ACCEPT;
iptables -A FORWARD -o wg0 -j ACCEPT;
```
