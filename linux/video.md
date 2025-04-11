# Contents

- [different monitor resolutions](#different-monitor-resolutions)
- [stucked VIRTUAL monitor](#stucked-virtual-monitor)
- [get window class or name](#get-window-class-or-name)
- [get iGPU memory](#get-igpu-memory)
- [touchscreen](#touchscreen)
- [codecs](#codecs)
- [DPI](#dpi)
    - [Xorg](#xorg)

# different monitor resolutions
e.g.
EXT_DISPLAY - `2560x1080`
INT_DISPLAY - `1920x1200`

EXT_DISPLAY renders everything too big.
Workaround is to scale up its resolution (e.g. in `1.5`)
```
xrandr --output EXT_DISPLAY --scale 1.5x1.5 --mode 2560x1080 --fb 3840x2820 --pos 0x0
xrandr --output INT_DISPLAY --scale 1x1 --mode 1920x1200 --pos 0x1620
```
* `--fb` container for both screens
* `--pos` position from top-left corner

# stucked VIRTUAL monitor
* `xrandr --listmonitors` to find out which output is still in use (e.g. `HDMI1`)
* `xrandr --output <NAME> --off` 
  
# get window class or name
* `xprop | grep -i wm_class`
* `xprop | grep -i wm_name`


# get iGPU memory
- `lspci` to get VGA domain
- `lspci -v -s <domain>` where `<domain>` is e.g. `00:02.0`


# touchscreen
* `xinput --list` to get names of touch controllers, smth with **stylus** and **eraser** is required
* `xinput --map-to-output '<NAME>' $DISPLAY` map to display


# codecs
| Device          | WebRTC      | MSE         | MP4         |
|-----------------|-------------|-------------|-------------|
| latency         | best        | medium      | bad         |
| Desktop Chrome  | H264        | H264, H265* | H264, H265* |
| Desktop Safari  | H264, H265* | H264        | no          |
| Desktop Edge    | H264        | H264, H265* | H264, H265* |
| Desktop Firefox | H264        | H264        | H264        |
| Desktop Opera   | no          | H264        | H264        |
| iPhone Safari   | H264, H265* | no          | no          |
| iPad Safari     | H264, H265* | H264        | no          |
| Android Chrome  | H264        | H264        | H264        |
| masOS Hass App  | no          | no          | no          |


# DPI

## Xorg
* add to `/etc/X11/xorg.conf.d/90-monitor.conf`:
  ```
  Section "Monitor"
    Identifier             "<default monitor>"
    DisplaySize            189  118    # In millimeters
  EndSection
  ```
* add `Xft.dpi: 192` to `Xresources` and execute `xrdb -merge <path/to/Xresources>`
