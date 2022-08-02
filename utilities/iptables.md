# Contents

- [configuration](#configuration)
- [add rule](#add rule)
- [list rules](#list rules)
- [delete rule](#delete rule)
- [flush rules](#flush rules)
- [recepies](#recepies)
    - [open port](#recepies#open port)
    - [port forwarding](#recepies#port forwarding)
    - [trafic forwarding](#recepies#trafic forwarding)
    - [basic routing between eth0 and wlan0](#recepies#basic routing between eth0 and wlan0)

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
