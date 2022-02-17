# Contents

- [setup](#setup)
- [troubleshooting](#troubleshooting)
    - [ssl verification](#troubleshooting#ssl verification)

# setup
* allow forwarding
```bash
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
sysctl -w net.ipv4.conf.all.send_redirects=0
```
* forward ports
```bash
iptables -t nat -A PREROUTING -i <interface> -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -i <interface> -p tcp --dport 443 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -i <interface> -p tcp --dport 80 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -i <interface> -p tcp --dport 443 -j REDIRECT --to-port 8080
```
* run mitm
`mitmproxy -m transparent --showhost`
* set DHCP address in Wi-Fi settings on target device to IP of machine with `mitmproxy`

# troubleshooting
## ssl verification
SSL verification must be turned off in order to use mitm proxy

```python
class Client:
    def __init__(self):
        self.session.verify = False
```
