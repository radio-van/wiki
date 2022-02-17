# Contents

- [software](#software)
- [usecases](#usecases)
    - [store alsa settings](#usecases#store alsa settings)
    - [mute/unmute](#usecases#mute/unmute)
    - [get volume in %](#usecases#get volume in %)
    - [get current output device](#usecases#get current output device)
- [bluetooth](#bluetooth)
    - [switch profiles](#bluetooth#switch profiles)

# software
cli:
`amixer` useful for selecting cards, devices
`pamixer` useful to get/set current volume, switch sink/source
tui:
`alsamixer` from `alsa-utils`
`pulsemixer` (like `alsamixer`, only for pulseaudio)

configuration:
`alsactl`, `pactl`

noise filter:
`noisetorch`

# usecases
## store alsa settings
`sudo alsactl store`

## mute/unmute
`pamixer -t`

## get volume in %
`pamixer --get-volume-human`

## get current output device
`pactl info`

# bluetooth
## switch profiles
Supported profiles can be viewed with `pacm list-cards`, along with `<ID>`.  
* `pacmd set-card-profile <ID> a2dp_sink` A2DP profile
* `pacmd set-card-profile <ID> headset_hed_unit` headset profile (with mic support)

Restrict auto switching:  
`/etc/pulse/default.pa`
```
load-module module-bluetooth-policy auto_switch=false
```

Automatically switch to newly-connected devices:  
`/etc/pulse/default.pa`
```
load-module module-switch-on-connect
```

Also:  
`/etc/pulse/default.pa`
```
load-module module-bluetooth-discover
```
