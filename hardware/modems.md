## Contents =
    - [[#firmware|firmware]]
        - [[#firmware#Huawei 827F|Huawei 827F]]
            - [[#firmware#Huawei 827F#recovery mode|recovery mode]]
            - [[#firmware#Huawei 827F#flashing|flashing]]
        - [[#firmware#ZTE modem 836F|ZTE modem 836F]]
            - [[#firmware#ZTE modem 836F#flashing|flashing]]
            - [[#firmware#ZTE modem 836F#modes|modes]]
            - [[#firmware#ZTE modem 836F#adb shell configuration|adb shell configuration]]
    - [[#AT commands|AT commands]]
        - [[#AT commands#overview|overview]]
        - [[#AT commands#IMEI|IMEI]]
        - [[#AT commands#SMS|SMS]]

## firmware =
### Huawei 827F ==
There are two revisions of modems:
* Е3372 *S* with *Stick* firmware (2 rows of contacts)
* Е3372 *H* with *Hilink* firmware (1 row of contacts)

#### recovery mode ===
Short *boot pin* with metal plate of *USB connector*
* E3372S *boot pin* is the first pin in top row
* E3372H *boot pin* is the most right in a row

#### flashing ===
From recovery mode:
- upload bootloader `balong-usbdload -p <port> <bootloader>`
*WARNING* use custom bootloader (*balong* [[https://github.com/forth32/balongflash|usbsafe-3372X]]) otherwise _nvram_ would be lost

From noraml mode:
- switch to flashing composition
    - *Stick* `echo -e "at^godload\r" >/dev/ttyUSB0`
    - *Hlink* `adb shell 'echo -e "at^godload\r" >/dev/appvcom1'` or `echo -e "at^godload\r" >/dev/appvcom1` via *Telnet*
- modem will reconnect itself in flashing composition
    - *Stick* 3 USB ports, the highest is for firmware
    - *Hlink* 2 USB ports, the lowest is for firmware
- `./balong_flash -p <port> <firmware>`
  use `-k` key if you flash *firmware* and *webui* separately
  use `-r` key if modem doen't return to normal mode after flashing

### ZTE modem 836F ==
*Windows* software
* `terminal` *COM* terminal for sending *AT* commands
* `SCSI.exe` switches modem mode (see [[#modes|below]]) to enable firmware update
* `ZTE Software` firmware tool

#### flashing ===
- run `SCSI` to switch modem to appropriate mode
- run `ZTE Software`, flash `unlock`, then `firmware` (disconnect modem between flashing)
- using terminal, switch to prefered [[#modes|mode]]

#### modes ===
* *MODE 0* works as modem + optical drive
* *MODE 1* works as modem + `adb`
* *MODE 2* firmware ready

#### adb shell configuration ===
more info about [[../utilities/adb#Contents|adb]]
`cfg show` show config
`cfg set <property>=<value>` change configuration
`cfg save` save configuration


## AT commands =
### overview ==
To use *AT* commands on Linux try
`socat - /dev/ttyUSB2,crnl`
or `minicom`

use `ate1` command to turn on echo

### IMEI ==
`at^modimei=<imei>`
or
`at^setimei=<imei>`

### SMS ==
`AT+CMGR=<index>` read SMS
`AT+CMGL="REC UNREAD"` list unread SMS
`AT+CMGL="ALL"` list all SMS
