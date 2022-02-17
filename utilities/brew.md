# Contents

- [receipts](#receipts)
    - [downgrade package](#downgrade-package)
    - [prevent package from upgrading](#prevent-package-from-upgrading)
    - [multi-user system](#multi-user-system)
    - [show dependency tree](#show-dependency-tree)

# receipts
## downgrade package

`brew search <package name>`   

Older/specific versions are marked with `@`   
```
    brew install <package>@<version>
    brew unlink <old package>
    brew link --force --overwrite <new package>
```

Also it's a good idea to [prevent package from upgrading](#prevent package from upgrading)

## prevent package from upgrading
`brew pin <package>`

## multi-user system
```
    chgrp -R admin /usr/local/*
    chmod -R g+w /usr/local/*
```

## show dependency tree
`brew deps --tree --installed`
