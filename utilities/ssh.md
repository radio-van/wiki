# Contents

- [configuration](#configuration)
    - [disabling password auth](#configuration#disabling password auth)
    - [disabling root login](#configuration#disabling root login)
    - [do not ask about adding host keys](#configuration#do not ask about adding host keys)
    - [solving SSH-RSA](#configuration#solving SSH-RSA)
    - [limit login access](#configuration#limit login access)
    - [set shell](#configuration#set shell)
    - [override settings](#configuration#override settings)
- [usage](#usage)
    - [connection](#usage#connection)
        - [force using password](#usage#connection#force using password)
        - [force using IdentityOnly](#usage#connection#force using IdentityOnly)
    - [keys](#usage#keys)
        - [keygen](#usage#keys#keygen)
        - [copy key](#usage#keys#copy key)
        - [generate public key from private](#usage#keys#generate public key from private)
        - [storing keys](#usage#keys#storing keys)
        - [key fingerprint](#usage#keys#key fingerprint)
- [ssh-agent](#ssh-agent)
    - [overview](#ssh-agent#overview)
    - [usage](#ssh-agent#usage)
    - [agent-forwarding](#ssh-agent#agent-forwarding)
        - [basics](#ssh-agent#agent-forwarding#basics)
        - [configuration](#ssh-agent#agent-forwarding#configuration)
- [port forwarding](#port forwarding)
- [proxy command](#proxy command)
- [scp](#scp)
    - [ambiguous target](#scp#ambiguous target)
- [additional security](#additional security)

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


# additional security
- **fail2ban**
- [endlessh](https://github.com/skeeto/endlessh)
