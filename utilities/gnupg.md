# Contents

- [configuration](#configuration)
- [usage](#usage)
    - [generate](#usage#generate)
    - [list](#usage#list)
    - [edit](#usage#edit)
    - [export](#usage#export)
    - [import](#usage#import)
    - [delete](#usage#delete)
    - [encrypt/decrypt](#usage#encrypt/decrypt)
        - [use multiple keys](#usage#encrypt/decrypt#use multiple keys)
    - [git integration](#usage#git integration)
    - [edit uid](#usage#edit uid)
- [gpg-agent](#gpg-agent)
    - [configuration](#gpg-agent#configuration)
    - [restart](#gpg-agent#restart)
    - [ssh integration](#gpg-agent#ssh integration)
    - [compare ssh key with GPG keygrip](#gpg-agent#compare ssh key with GPG keygrip)
        - [SSH key > GPG key](#gpg-agent#compare ssh key with GPG keygrip#SSH key > GPG key)
        - [GPG key > SSH key](#gpg-agent#compare ssh key with GPG keygrip#GPG key > SSH key)
    - [import existing key](#gpg-agent#import existing key)
    - [forget passphrase](#gpg-agent#forget passphrase)
- [troubles](#troubles)
    - [gpg-agent from tmux](#troubles#gpg-agent from tmux)

    GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management
    system, along with access modules for all kinds of public key directories.
    GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications.
    A wealth of frontend applications and libraries are available.
    
# configuration
default storage is `~/.gnupg` and can be modified with `$GNUPGHOME` env variable  
default config files are `~/.gnupg/gpg.conf` and `~/.gnupg/dirmngr.conf`  
temporary storage path can be modified with `--homedir` option  

`~/.gnupg/gpg-agent.conf` handles `gpg-agent` preferences, e.g. `enable-ssh-support` to emulate `ssh-agent`  
`~/.gnupg/sshcontrol` stores keys for use with `ssh`  

Keys identities could be:
- *hash*  - the hash of key in different formats, e.g. `0xlong`. Represents only particular (*public* or *private*) key
- *key ID* - aka *fingerprint*
- *keygrip* refers to both *private* and *public* keys in keypair. Must be used in `sshcontrol` file. Could be obtained by `--with-keygrip` option.

useful config
```
    keyid-format 0xlong  # more robust IDs
    throw-keyids         # do not include key info into encrypted message
    no-emit-version      # do not include pgp version
    no-comments          # do not include comments
    with-keygrip
    with-subkey-fingerprint
```
# usage
## generate
generate key `gpg --full-gen-key --expert`
generate revocation certificate `gpg --gen-revoke --armor --output=revocation_certificate.asc user-id`
for *ssh* additional *A* - *Auth* capability must be set for the key

tip: for passphrase generation a dictionary + `shuf` could be used:
`shuf --random-source=/dev/random -n 6 ./large_wordlist.txt`

## list
`gpg --list-keys`
`gpg --list-secret-keys`
add `--with-keygrip` to show *keygrip*

`rsa|dsa|ed25519` — crypto algorithm
`2048` — key length
_second line_ - key fingerprint (must be verified with owner)
`uid` — User-ID
`pub|sub|sec|ssb` — key type:
    `pub` — public key
    `sub` — public subkey
    `sec` — private key
    `ssb` — private subkey
[XX] — key application:
    S — signing
    C — certification aka key signing
    E — encryption
    A — authentication (for e.g. *ssh*)
    
## edit
`gpg --edit-key <user-id>`

## export
private `gpg --export-secret-keys --armor <user-id> > privkey.asc`
public `gpg --output public.key --armor --export user-id`

use `<keyid>!` to export particular subkey  
`<keyid>` can be found with `--with-subkey-fingerprint` option  

tip: as QR code with `qrencode`:
`gpg --export-secret-key <keygrip> | qrencode --8bit --output secret-key.qr.png`

tip: export tiny private key as QR with `paperkey`:
*WARNING!* recovery such private key is possible only with public key
```
    gpg --export-key <keygrip> > pubkey.asc
    gpg --export-secret-key <keygrip> | paperkey --output-type raw | qrencode --8bit --output secret-key.qr.png
```
recovery:
```
    paperkey --pubring pubkey.asc --secrets from_qr.asc > secretkey.asc
    gpg --import ./secretkey.asc
```

## import
`gpg --import public.key`

## delete
`gpg --delete-secret-key <keygrip>`

## encrypt/decrypt
encrypt `gpg --recipient user-id --encrypt doc`
decrypt `gpg --output doc --decrypt doc.gpg`

### use multiple keys
`gpg --encrypt --recipient alice@example.com --recipient bob@example.com <file>`

## git integration
- export public key with `--armor` option
- git config
```
    [commit]
            gpgsign = true
    [user]
            signingkey = <KeyID>
    [gpg]
            program = /bin/gpg
```
* use `-S` option with `git commit` to sign
* use `git log --show-signature -1` to show signatures
* use `--verify-signatures` with `git pull` and `git merge`

## edit uid
This can be done via adding new uid
```
gpg ---edit-key <keyID>
> adduid
> uid <new uid>
> trust
> uid <old uid>
> revuid
> save
```

# gpg-agent

## configuration
`~/.gnupg/gpg-agent.conf`
```
default-cache-ttl 43200  # expire cache if it wasn't accessed for given seconds
default-cache-ttl-ssh 43200  # expire ssh keys if they weren't accessed for given seconds
max-cache-ttl 86400  # expire cache (even if it was accessed recently)
max-cache-ttl-ssh 86400  # expire cache for ssh keys (even if they were accessed recently)

enable-ssh-support  # act as ssh agent

pinentry-program /usr/bin/pinentry-dmenu  # program to enter passphrase

allow-preset-passphrase  # ? allows to cache passphrase
```

## restart
`gpg-connect-agent reloadagent /bye`  

## ssh integration
key must be generated with *Auth* capability
- add key [keygrip](#list) to `~/.gnupg/sshcontrol`
  good idea is to add `confirm` option after the key to secure AgentForwarding
- export key `gpg --export-ssh-key <keygrip>`
- export public key to github as *GPG* key `gpg --export --armor <keygrip>`
- restart *gpg-agent* `gpg-connect-agent reloadagent /bye`
- (optional) check connection `ssh -T git@github.com`

## compare ssh key with GPG keygrip
### SSH key > GPG key
* `ssh-add -L | head -1 > <key>` - ssh public key fingerprint
* `ssh-keygen -l -E md5 -f <key>` - MD5 fingerprint
* `gpg-connect-agent 'KEYINFO --list --ssh-fpr' /bye` - list GPG keys with MD5 fingerprints

### GPG key > SSH key
* `gpg -K --with-subkey-fingerprint` - list keys
* `gpg --export-ssh-key <subkey fingerprint>`
* at the end `opengpg: <ID>` can be used as identifier

## import existing key
Conversion from *PEM* (ssh) to *OpenPGP* (gpg) is needed.  
* `ssh-keygen -p -m PEM -f <private.keyfile>` - convert from newer OpenSSH format
* `gpg -a --export-secret-keys <identity?> > bkp` - backup
* `gpg --homedir <temp> --import bkp` - import backup to temp workdir. *NOTE* `<temp>` folder must have 700 permissions
* install `monkeyshare` for `pem2openpgp` utility
* `pem2openpgp temporary_id < .ssh/<key> | gpg --import --homedir <temp>`
* `gpg --homedir <temp> --expert --edit-key <identity>`
    * `addkey` -> 13 -> `<keygrip>` of newly added key
    * set *actions* to `Authenticate`
    * save
* `gpg --homedir <temp> -a --export-secret-keys <identity>` - export existing key with new SSH subkey
* `gpg --import bkp` import from temporary backup to *master* keyring (only *subkey* will be imported)
* add new key to `sshcontrol` file

## forget passphrase
`echo RELOADAGENT | gpg-connect-agent`


# troubles
## gpg-agent from tmux
error: `sign_and_send_pubkey: signing failed for ED25519 "(none)" from agent: agent refused operation`  
solution: *outside* tmux `gpg-connect-agent updatestartuptty /bye`  
