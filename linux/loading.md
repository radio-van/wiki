# Contents

- [Continue loading on init errors](#Continue loading on init errors)
- [LiveCD](#LiveCD)

# Continue loading on init errors
From `rootfs` shell.
- mount root partition
- `exec switch_root <root partition mount> /sbin/init`

# LiveCD
Has two file systems:
- `overlayfs` read only image
- `tmpfs` for writing
