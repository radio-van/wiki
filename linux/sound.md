# Contents

- [software](#software)
- [usecases](#usecases)
    - [store alsa settings](#store-alsa-settings)
    - [mute/unmute](#muteunmute)
    - [get volume in %](#get-volume-in-)
    - [get current output device](#get-current-output-device)
    - [output to all sinks](#output-to-all-sinks)
- [bluetooth](#bluetooth)
    - [switch profiles](#switch-profiles)

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

## output to all sinks
`pactl load-module module-combine-sink`  
for permanent: `.config/pipewire/pipewire-pulse.conf`
```context.exec = [
    { path = "pactl"  args = "load-module module-combine-sink" }
]
```
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
