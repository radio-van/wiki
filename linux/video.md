# Contents

- [different monitor resolutions](#different monitor resolutions)
- [stucked VIRTUAL monitor](#stucked VIRTUAL monitor)
- [get window class or name](#get window class or name)

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
