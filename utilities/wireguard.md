# Contents

- [basics](#basics)
- [configuration](#configuration)
    - [configs](#configs)
    - [command line](#command-line)
    - [key generation](#key-generation)
    - [NAT](#nat)
- [pfsense as wireguard server](#pfsense-as-wireguard-server)

# basics
    Each node has `private_key` and `public_key`.
    `public_key` is associated with [[../linux/networking#THEORY#interfaces#TUN/TAP|tunnel]] interface and used to auth clients. 

    `outgoing packet` for `internal_ip`
    ---> lookup `public_key` by ip from associations list
    ---> encrypt packet with `public_key`
    ---> lookup `external ip` by `public_key`
    ---> send packet to `external_ip:UDP_port`

    `incoming packet` ---> decrypt
    ---> lookup for peer ---> check peer auth
    ---> remember `external_ip` for peer
    ---> send plain packet to `internal_ip`
    ---> check if peer is allowed
 
 
# configuration

## configs
Here peers described as **server/client**, however in Wireguard they are all equal. 

**Server**
```bash
  [Interface]
  PrivateKey = ...
  ListenPort = 51820

  [Peer]
  PublicKey = ...
  AllowedIPs = 10.192.122.3/32  # internal Wireguard NET

  [Peer]
  PublicKey = ...
  AllowedIPs = 10.192.122.4/32, 192.168.0.0/16  # allow this peer to access to server's LAN
```

**Client**
```bash
  [Interface]
  PrivateKey = gI6EdUSYvn8ugXOt8QQD6Yc+JyiZxIhp3GInSWRfWGE
  ListenPort = 21841

  [Peer]
  PublicKey = HIgo9xNzJMWLKASShiTqIybxZ0U3wGLiUeJ1PKf8ykw
  Endpoint = <EXTERNAL IP>
  AllowedIPs = 0.0.0.0/0  # wildcard ip, allow any address, DANGEROUS
```

* when sending packets, the list of `allowed IPs` behaves as a routing table
* when receiving packets, the list of `allowed IPs behaves` as an access control list

    The client configuration contains an initial endpoint of its single peer (the server), so that it knows where to send encrypted data before it has received encrypted data.
    The server configuration doesn't have any initial endpoints of its peers (the clients), it discovers the endpoint of its peers by examining from where correctly authenticated data originates.
    If the server itself changes its own endpoint, and sends data to the clients, the clients will discover the new server endpoint and update the configuration just the same.
    
## command line

* create new interface
`ip link add dev wg0 type wireguard`
  * (Non-Linux users will instead write `wireguard-go wg0`)

* assign  IP address
`ip address add dev wg0 192.168.2.1/24`

* configure interface with keys and peer endpoints
    - `wg setconf wg0 myconfig.conf`
    or  
    - `wg set wg0 listen-port 51820 private-key /path/to/private-key peer ABCDEF... allowed-ips 192.168.88.0/24 endpoint 209.202.254.14:8172`

* activate interface
`ip link set up dev wg0`

OR  

* `wg-quick up /path/to/conf/file`


additional commands::
- `wg show`
- `wg showconf`

## key generation

WireGuard requires base64-encoded public and private keys.
```bash
  $ umask 077
  $ wg genkey > privatekey
``` 
This will create privatekey on stdout containing a new private key.
You can then derive your public key from your private key:
`$ wg pubkey < privatekey > publickey`

This will read privatekey from stdin and write the corresponding public key to publickey on stdout.

You can do this all at once:
`$ wg genkey | tee privatekey | wg pubkey > publickey`

## NAT
    Because NAT and stateful firewalls keep track of "connections", if a peer behind NAT  
    or a firewall wishes to receive incoming packets, he must keep the NAT/firewall mapping valid,  
    by periodically sending keepalive packets. This is called _persistent keepalives_.  
    When this option is enabled, a keepalive packet is sent to the server endpoint once every interval seconds.  
    A sensible interval that works with a wide variety of firewalls is *25* seconds.  
    Setting it to *0* turns the feature off, which is the default, since most users will not need this,  
    and it makes WireGuard slightly more chatty.  
    This feature may be specified by adding the `PersistentKeepalive =` field to a peer in the configuration file,  
    or setting `persistent-keepalive` at the command line.
    

One or more peers can act as a **router** with following **iptables** rules:
```
[Interface]
Address = 10.0.1.1/24
ListenPort = 51902
PrivateKey = ...

PostUp = iptables -I INPUT 1 -p udp --dport 51902 -j ACCEPT; iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o wg0 -j MASQUERADE
PostDown = iptables -D INPUT -p udp --dport 51902 -j ACCEPT; iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -s 10.0.1.0/24 -o wg0 -j MASQUERADE

[Peer]
PublicKey = ...
AllowedIPs = 10.0.1.2/24
PersistentKeepalive = 25

[Peer]
PublicKey = ...
AllowedIPs = 10.0.1.3/32
PersistentKeepalive = 25
```

In this case, peers should have the **router peer** described as following:
```
[Peer]
PublicKey = ...
AllowedIPs = 10.0.1.0/24
Endpoint = <EXTERNAL IP>
PersistentKeepalive = 25
```
And ignore all other peers.  
Setting `AllowedIPs = 10.0.1.0/24` allows all other peers to communicate via **router peer**.


# pfsense as wireguard server

* add port forwarding from `WAN` to `WG` iface (UDP, wg listen port, WG IP) and check _add associated filter rule_
* add wireguard firewall rule with _pass_ action, from WG iface for IPv4 from WG subnet source to Any
