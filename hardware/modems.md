# Contents

- [firmware](#firmware)
    - [Huawei 827F](#huawei-827f)
        - [recovery mode](#recovery-mode)
        - [flashing](#flashing)
        - [switch hilink to AT-mode](#switch-hilink-to-at-mode)
    - [ZTE modem 836F](#zte-modem-836f)
        - [flashing](#flashing-2)
        - [modes](#modes)
- [AT commands](#at-commands)
    - [overview](#overview)
    - [IMEI](#imei)
    - [SMS](#sms)
- [recovering](#recovering)

# firmware
## Huawei 827F
There are two revisions of modems:
* Е3372 *S* with *Stick* firmware (2 rows of contacts)
* Е3372 *H* with *Hilink* firmware (1 row of contacts)

### recovery mode
Short *boot pin* with metal plate of *USB connector*
* E3372S *boot pin* is the first pin in top row
* E3372H *boot pin* is the most right in a row

### flashing
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

### switch hilink to AT-mode
`http://192.168.8.1/html/switchProjectMode.html`  
*NOTE*: if modem is not shown as Ethernet card, `usb_modeswitch` must be installed.  

Modem should be now accessable as `/dev/ttyUSBX`

or use [4pda script](https://4pda.ru/forum/index.php?showtopic=582284&st=39980#entry76659248)

## ZTE modem 836F
*Windows* software
* `terminal` *COM* terminal for sending *AT* commands
* `SCSI.exe` switches modem mode (see [[#modes|below]]) to enable firmware update
* `ZTE Software` firmware tool

### flashing
- run `SCSI` to switch modem to appropriate mode
- run `ZTE Software`, flash `unlock`, then `firmware` (disconnect modem between flashing)
- using terminal, switch to prefered [[#modes|mode]]

### modes
* *MODE 0* works as modem + optical drive
* *MODE 1* works as modem + `adb`
* *MODE 2* firmware ready

# AT commands
## overview
To use *AT* commands on Linux try
`socat - /dev/ttyUSBX,crnl`
or `minicom /dev/ttyUSBX`

use `ate1` command to turn on echo  
or type `Ctrl+A E` in `minicom`

type `AT^CURC=0` in `minicom` to turn off data flow from modem.

## IMEI
**IMEI** has first 8 meaningful digits and 6 random.
For *Nokia* (because it's the easiest way to provide proper modem work on Windows PC),
first digits would be `35365206`.
The rest 6 are random and can be checked at [imei.info](http://www.imei.info/calc).

**IMEI** must be written into hardware via AT commands.

`at^modimei=<imei>`
or
`at^setimei=<imei>`

For **E3372h**:  
`AT^CIMEI="<imei>"`
`AT^INFORBU`

more info: [4pda](https://4pda.ru/forum/index.php?showtopic=582284&view=findpost&p=51716190)
IMEI generator: [nokia](https://www.nokiaport.de/tacdatabase/index.php?s=imeitools&lng=)

## SMS
`AT+CMGR=<index>` read SMS
`AT+CMGL="REC UNREAD"` list unread SMS
`AT+CMGL="ALL"` list all SMS

# recovering
Most often case is corrupted flash.

Install drivers for **adb**, **fc serial**, **huawei**

- activate recovery mode with _pin_
- remap bad blocks with `usbdload` from **4pda**
- flash `usbsafe` from **balong**

Now modem should be in composition with single COM port (COM3)

- use **dc-...** to switch composition to `PC UC COM` (two COM ports)
- use **balong flash** or _official_ flasher to install firmware
- flash WEBUI
