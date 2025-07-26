# Contents

- [configuration](#configuration)
    - [disabling password auth](#disabling-password-auth)
    - [disabling root login](#disabling-root-login)
    - [do not ask about adding host keys](#do-not-ask-about-adding-host-keys)
    - [solving SSH-RSA](#solving-ssh-rsa)
    - [limit login access](#limit-login-access)
    - [set shell](#set-shell)
    - [override settings](#override-settings)
    - [listen to specific interface](#listen-to-specific-interface)
- [usage](#usage)
    - [connection](#connection)
        - [force using password](#force-using-password)
        - [force using IdentityOnly](#force-using-identityonly)
    - [keys](#keys)
        - [keygen](#keygen)
        - [copy key](#copy-key)
        - [generate public key from private](#generate-public-key-from-private)
        - [storing keys](#storing-keys)
        - [key fingerprint](#key-fingerprint)
        - [remove server fingerprint from known_hosts](#remove-server-fingerprint-from-known_hosts)
- [ssh-agent](#ssh-agent)
    - [overview](#overview)
    - [usage](#usage-2)
    - [agent-forwarding](#agent-forwarding)
        - [basics](#basics)
        - [configuration](#configuration-2)
- [port forwarding](#port-forwarding)
- [proxy command](#proxy-command)
- [scp](#scp)
    - [ambiguous target](#ambiguous-target)
    - [no sftp (e.g. in ash)](#no-sftp-eg-in-ash)
- [additional security](#additional-security)
- [troubleshooting](#troubleshooting)
    - [stuck at debug1: expecting SSH2_MSG_KEX_ECDH_REPLY](#stuck-at-debug1-expecting-ssh2_msg_kex_ecdh_reply)

# configuration

## disabling password auth
`/etc/ssh/sshd_config`
uncomment: `PasswordAuthentication no`
apply: `service ssh restart`

## disabling root login
`/etc/ssh/sshd_config`
uncomment: `PermitRootLogin no`
apply: `service ssh restart`

## do not ask about adding host keys
`~/.ssh/config`
```
Host *
  StrictHostKeyChecking no
```

## solving SSH-RSA
**error**: `Unable to negotiate with <address>: no matching host key type found. Their offer: ssh-rsa`

add to `~/.ssh/config`:
```
Host <host>
    ...
    PubkeyAcceptedAlgorithms +ssh-rsa
    HostkeyAlgorithms +ssh-rsa
    ...
```

## limit login access
`/etc/ssh/sshd_config`
add: `AllowUsers user1 user2 ...`
apply: `service ssh restart`

## set shell
to restrict user to `git` only:  
`usermod -s /usr/bin/git-shell <username>`

## override settings
`/etc/ssh/sshd_config`
```
Match Address <ip/mask>
  <setting>
```

## listen to specific interface
`/etc/ssh/sshd_config`
```
ListenAddress <ip>
```
If `<ip>` is e.g. Wireguard interface, **sshd** should wait until it's active:
`/ets/systemd/system/sshd.service`
```
After=network.target wg-quick@wg0.service
Requires=sys-devices-virtual-net-wg0.device
```


# usage

## connection
### force using password
`ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no $host`
### force using IdentityOnly
When using `gpg-agent` instead of `ssh-agent` it's required to explicitly use IdentityFile:  
`ssh -o IdentitiesOnly=yes -i <identity_file> <user>@<host>`

`gpg-agent` will ask for passphrase to securely store this identity

## keys

### keygen
`ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f FILE`
`ssh-keygen -t ed25519 -C "your_email@example.com" -f FILE`

### copy key
`ssh-copy-id -i $FILE $user@$host`

### generate public key from private
`ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.pub` (comment would be lost)

### storing keys
`chmod 700 ~/.ssh`
`chmod 600 ~/.ssh/authorization_keys`

### key fingerprint
```
ssh-keyscan -t rsa github.com | ssh-keygen -lf -
```
useful for compare with public key fingerprint, e.g. for [github](https://docs.github.com/en/github/authenticating-to-github/githubs-ssh-key-fingerprints)

### remove server fingerprint from known_hosts
`ssh-keygen -R <hostname/IP>`


# ssh-agent

## overview
    The most secured area for storing private key's passphrase is RAM. Due to temporary nature of
    ssh client process, user has to type passphrase each time ssh client is launched. Idea is to store
    unencrypted private key in memory area of *ssh-agent* and communicate with it via socket in `/tmp`.
    *Attention!* Root user can read socket, so root user must be trustworthy.
    Socket is available via `$SSH_AGENT_SOCK`.
    
## usage
* `ssh-add <key>` add key, add `-c` option to require confirmation each time key would be read
* `ssh-add -l` list added keys

## agent-forwarding

### basics
    `ssh-agent forwarding` allows to use local `ssh-agent` on client machine as it is local for host machine.
    i.e. if user connects from machine *A* to machine *B* via intermediate server *S*, it's not necessary that
    server's key is authorized on machine *B*. With agent-forwarding *A*'s key must be authorized on *B* and *S*
    and *S* can has no access to *B*.
    However, while such connection is active, there is an opened socket, that allows `ssh-client` on server *S*  
    to connect to `ssh-agent` on host *A*, i.e. anyone with sufficient rights on server *S* can use it.  
```
   
      +-------key A----------------------+
      |                                  \/
   client A -------> server S -------> host B
   key A             auth: key A       auth: key A

```

### configuration
* server side `AllowAgentForwarding yes`
* client side `ForwardAgent yes`
* autostart `eval ssh-agent` (`eval` is used due to exporting env variable)
* `SSH_ASKPASS` determines which program would be used for asking user's confirmation for key reading
    
    
# port forwarding
* `-N` prevents opening remote shell


# proxy command
Specifies a *ProxyCommand* to connect to specified host.  
In general it could be any command. Useful for more secure than [agent-forwarding](#agent-forwarding)  
connection to host **B** which is accessible only from host **A**.  
```
Host A
  User ...
  Hostname ...
  
Host B
  User ...
  Hostname ...
  ProxyCommand ssh -q -W %h:%p A
```
Connection to host **B** will do a connection to host **A** first and then forwards ssh-client to `host:port`  
defined by `-W` option (i.e. to host and port of `B`).  
`-W` option forwards *stdin* and *stdout* to defined host via secure channel. It implies `-T` and `-N`,  
the last one prevents executing remote command (i.e. shell) and just forwards ports.  
Therefore, there is no classic ssh connection to host `A`, it used only for port-forwarding (in this  
case, *stdin/stdout* forwarding).  


# scp

## ambiguous target

All spaces in target path must be escaped with `\\` and path itself must be enclosed with `"`

## no sftp (e.g. in ash)
use legacy protocol `scp -O ...`


# additional security
- **fail2ban**
- [endlessh](https://github.com/skeeto/endlessh)


# troubleshooting

## stuck at debug1: expecting SSH2_MSG_KEX_ECDH_REPLY
add `-o KexAlgorithms=ecdh-sha2-nistp521`
