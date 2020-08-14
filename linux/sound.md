# Contents

- [software](#software)
- [usecases](#usecases)
    - [store alsa settings](#store-alsa-settings)
    - [mute/unmute](#muteunmute)
    - [get volume in %](#get-volume-in-)
    - [get current output device](#get-current-output-device)

# software
cli:
`amixer` useful for selecting cards, devices
`pamixer` useful to get/set current volume, switch sink/source
tui:
`alsamixer` from `alsa-utils`
`pulsemixer` (like `alsamixer`, only for pulseaudio)

configuration:
`alsactl`, `pactl`

# usecases
## store alsa settings
`sudo alsactl store`

## mute/unmute
`pamixer -t`

## get volume in %
`pamixer --get-volume-human`

## get current output device
`pactl info`
