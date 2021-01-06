# Contents

- [installation](#installation)
- [configuration =](#configuration-)
- [usage](#usage)
    - [add new password](#add-new-password)
    - [generate new password](#generate-new-password)
    - [retrieve password](#retrieve-password)
- [sync via git](#sync-via-git)

    With pass, each password lives inside of a gpg encrypted file whose filename is the title of the website or resource that requires the password.
    These encrypted files may be organized into meaningful folder hierarchies, copied from computer to computer, and, in general, manipulated using standard command line file management utilities.
   
# installation
as an addition to `pass` package, `gnupg` package must be installed and configured
# configuration = 
init password store `pass init <gpg-id or email>`

# usage
## add new password
`pass insert descriptive/hierarchy/name`
`pass insert --multiline` to provide *username* on the second line
```
    Password Store
    └── descriptive
        └── hierarchy
            └── name
```
## generate new password
`pass generate path/to/name n`, where `n` is a password length

## retrieve password
`pass /path/to/name`
`pass -c /path/to/name` copy to clipboard if `xclip` is installed
`passmenu` use integration with `dmenu`

# sync via git
* Create local password store
`pass init <gpg key id>` assume [gnupg](gnupg.md) key pair is configured
* Enable management of local changes through Git
`pass git init`
* Add the the remote git repository as 'origin'
`pass git remote add origin user@server:~/.password-store`
* Push your local Pass history
`pass git push -u --all`
* Use `pass git pull/push` to sync password storage
