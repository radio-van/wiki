# Contents

- [usage](#usage)
    - [list devices](#list-devices)
    - [connect](#connect)
- [recepies](#recepies)
    - [airplane mode](#airplane-mode)
    - [cellular data](#cellular-data)
    - [wifi](#wifi)
    - [list global settings](#list-global-settings)
    - [change network mode](#change-network-mode)
    - [restart services](#restart-services)

# usage
## list devices
`adb devices`, use `-l` for additional info
## connect
`adb shell`, use `-s <id>` to connect to particular device

# recepies
## airplane mode
* enable (`[]` part is optional)
```bash
    adb shell settings put global airplane_mode_on 1
    adb shell am broadcast -a android.intent.action.AIRPLANE_MODE [--ez state true]
```
* disable (`[]` part is optional)
```bash
    adb shell settings put global airplane_mode_on 0
    adb shell am broadcast -a android.intent.action.AIRPLANE_MODE [--ez state false]
```
also try
`adb shell su -c '...'`

## cellular data
* enable `adb shell svc data enable`
* disable `adb shell svc data disable`

## wifi
* enable `adb shell svc wifi enable`
* disable `adb shell svc wifi disable`

## list global settings
`adb shell settings list global`

## change network mode
`adb shell settings put global preferred_network_mode[num] 1`
`adb shell stop ril-daemon`
`adb shell start ril-daemon`

## restart services
`am startservice com.xxx/.service.XXXService`
`am stopservice com.xxx/.service.XXXService`
