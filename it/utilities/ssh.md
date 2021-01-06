# Contents

- [configuration](#configuration)
    - [disabling password auth](#disabling-password-auth)
    - [disabling root login](#disabling-root-login)
    - [do not ask about adding host keys](#do-not-ask-about-adding-host-keys)
    - [limit login access](#limit-login-access)
- [usage](#usage)
    - [connection](#connection)
        - [force using password](#force-using-password)
    - [keys](#keys)
        - [keygen](#keygen)
        - [copy key](#copy-key)
        - [generate public key from private](#generate-public-key-from-private)
        - [storing keys](#storing-keys)
        - [key fingerprint](#key-fingerprint)
- [ssh-agent](#ssh-agent)
    - [overview](#overview)
    - [usage](#usage-2)
    - [agent-forwarding](#agent-forwarding)
        - [overview](#overview-2)
        - [configuration](#configuration-2)
- [scp](#scp)
    - [ambiguous target](#ambiguous-target)
- [additional security](#additional-security)

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

## limit login access
`/etc/ssh/sshd_config`
add: `AllowUsers user1 user2 ...`
apply: `service ssh restart`

# usage

## connection
### force using password
`ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no $host`

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
### overview
    `ssh-agent forwarding` allows to use local `ssh-agent` on client machine as it is local for host machine.
    i.e. if user connects from machine *A* to machine *B* via intermediate server *S*, it's not necessary that
    server's key is authorized on machine *B*. With agent-forwarding *A*'s key must be authorized on *B* and *S*
    and *S* can has no access to *B*.
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
    
# scp
## ambiguous target
All spaces in target path must be escaped with `\\` and path itself must be enclosed with `"`

# additional security
- **fail2ban**
- [endlessh](https://github.com/skeeto/endlessh)

