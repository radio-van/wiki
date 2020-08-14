# Contents

- [configuration](#configuration)
- [usage](#usage)
    - [generate](#generate)
    - [list](#list)
    - [edit](#edit)
    - [export](#export)
    - [import](#import)
    - [delete](#delete)
    - [encrypt/decrypt](#encryptdecrypt)
    - [git integration](#git-integration)
- [gpg-agent](#gpg-agent)
    - [configuration](#configuration-2)
    - [ssh integration](#ssh-integration)

    GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management
    system, along with access modules for all kinds of public key directories.
    GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications.
    A wealth of frontend applications and libraries are available.
    
# configuration
default storage is `~/.gnupg` and can be modified with `$GNUPGHOME` env variable
default config files are `~/.gnupg/gpg.conf` and `~/.gnupg/dirmngr.conf`

`~/.gnupg/gpg-agent.conf` handles `gpg-agent` preferences, e.g. `enable-ssh-support` to emulate `ssh-agent`
`~/.gnupg/sshcontrol` stores keys for use with `ssh`

Keys identities could be:
- *hash* or *key ID*? - the hash of key in different formats, e.g. `0xlong`. Represents only particular (*public* or *private*) key
- *keygrip* refers to both *private* and *public* keys in keypair. Must be used in `sshcontrol` file. Could be obtained by `--with-keygrip` option.

useful config
```
    keyid-format 0xlong  # more robust IDs
    throw-keyids         # do not include key info into encrypted message
    no-emit-version      # do not include pgp version
    no-comments          # do not include comments
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


# gpg-agent
## configuration
`~/.gnupg/gpg-agent.conf`
* program for entering passphrase `pinentry-program /usr/bin/pinentry-tty`
* act as ssh agent `enable-ssh-support `

## ssh integration
key must be generated with *Auth* capability
- add key [keygrip](#list) to `~/.gnupg/sshcontrol`
- export key `gpg --export-ssh-key <keygrip>`
- export public key to github as *GPG* key `gpg --export --armor <keygrip>`
- restart *gpg-agent* `gpg-connect-agent reloadagent /bye`
- (optional) check connection `ssh -T git@github.com`
