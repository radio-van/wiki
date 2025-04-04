# Contents

- [hardware related](#hardware-related)
    - [hardware info](#hardware-info)
    - [laptop lid](#laptop-lid)
    - [power buttons and lid](#power-buttons-and-lid)
    - [acpi rules](#acpi-rules)
        - [laptop lid hook](#laptop-lid-hook)
- [hibernation](#hibernation)
    - [swapfile](#swapfile)
- [os release](#os-release)

# hardware related

## hardware info
* `lshw -C memory|display|etc` (spare package)
* `lscpu` (part of `util-linux`)
* `dmidecode -t memory` (spare package)
* `lsgpu`
* `glxwinfo` (part of `glew`)

## laptop lid
status: `/proc/acpi/button/lid/LID/state`

## power buttons and lid
config behavior: `/etc/systemd/logind.conf`

## acpi rules
### laptop lid hook
```
button/lid)
  case "$3" in
    close)
      logger 'LID closed'
      /etc/acpi/scripts/lid-close &
      ;;
  open)
      logger 'LID opened'
      /etc/acpi/scripts/lid-open &
      ;;
```

# hibernation
* initramfs hooks should be configured: `HOOKS=(base udev autodetect keyboard modconf block filesystems `**resume**` fsck)`
*NOTE:* `resume` hook must be after `lvm2` if **LVM** is used  

## swapfile
* boot options must be added: `resume=/path/to/partition/with/swapfile resume_offset=<N>`
    * offset can be obtained: `filefrag -v /swapfile | awk '{ if($1=="0:"){print substr($4, 1, length($4)-2)} }'`
    * `resume=` should point to the required partition the same way as `root=` does, i.e. path to unencrypted volume if `LUKS` is used


# os release
`cat /etc/issue`
