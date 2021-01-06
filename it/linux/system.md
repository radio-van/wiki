# Contents

- [hardware related](#hardware-related)
    - [laptop lid](#laptop-lid)
    - [power buttons and lid](#power-buttons-and-lid)
    - [acpi rules](#acpi-rules)
        - [laptop lid hook](#laptop-lid-hook)

# hardware related
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
