# Contents

- [Virtual console](#Virtual console)
- [Xorg](#Xorg)
    - [keycodes](#Xorg#keycodes)
    - [modifiers](#Xorg#modifiers)
    - [xcape](#Xorg#xcape)
    - [xkb](#Xorg#xkb)
        - [options](#Xorg#xkb#options)
        - [frequently used XKB options](#Xorg#xkb#frequently used XKB options)
        - [one-click key functions](#Xorg#xkb#one-click key functions)
    - [Xmodmap](#Xorg#Xmodmap)
        - [modify keymap](#Xorg#Xmodmap#modify keymap)
        - [modifier keys](#Xorg#Xmodmap#modifier keys)
        - [reverse scrolling](#Xorg#Xmodmap#reverse scrolling)
        - [swap mouse buttons](#Xorg#Xmodmap#swap mouse buttons)
        - [disable touchpad](#Xorg#Xmodmap#disable touchpad)
- [backlight](#backlight)
    - [mackbook](#backlight#mackbook)
- [leds](#leds)

# Virtual console
`/etc/vconsole.conf`  
- install `terminus-font` for Unicode support
- set layout from `/usr/share/kbd/keymaps/i386/qwerty`, some of them has a toggle option
E.g.  
```
FONT=ter-u18n
KEYMAP=ruwin_cplk-UTF-8
```

# Xorg
## keycodes
`xev-xorg` can be used to get *key codes*.  

**keysum** = **keycode** + **state**, where:
- **state** is a modifier (e.g. *Shift*)
- **keycode** is sent to **X** from *device*
- **keysum** is what is sent to *XKeyEvent*, where it is obtained by *client* via *XLookupString*  

## modifiers
Keypresses in **X** have 8 modifiers bits: `Shift, Lock, Control, Mod1-Mod5`  
Both `Shift` *keys* are bound to `Shift` *modifier*, as `Ctrl` *keys* are bound to `Control` *modifier*.  
`Alt` *key* is usually `Mod1` *modifier*.

**X** still has internal keycodes for `Super`, `Meta` and `Hyper` keys from *Symbolics* keyboard.  
`Super` is usually `Mod4`.  

## xcape
Allows to map additional symbols on keys depending on their state - pressed or held.  
E.g.  
`xcape -e 'Shift_l=parenleft;Shift_R=parentright'`  
- `-t` param controls timeout

## xkb
works at *XLookupString* level and transforms incoming *keycodes* into *keysums* according to defined rules  

### options
Options are made in several sets:
* `XkbModel` selects keyboard model, has influence only on few additional extra keys
* `XkbLayout` selects layout, multiple can be set with comma
* `XkbVariant` selects layout variant (qwerty, dvorak, mac, etc), quantity must be the same with quantity of *layouts*, defaults can be omited (i.e just comma)
* `XkbOptions` sets variety of additional options, comma-separated

Available options are listed in `/usr/share/X11/xkb/rules/base.lst`.
Alternatively, following commands can be used:
* `localectl list-x11-keymap-models`
* `localectl list-x11-keymap-layouts`
* `localectl list-x11-keymap-variants [layout]`
* `localectl list-x11-keymap-options`

All those settings can be set:
* with `setxkbmap`, e.g. (Alt+Shift for switch layout, MacOS layout for Russian):
`setxkbmap -model pc104 -layout us,ru -variant ,mac -option grp:alt_shift_toggle`
* with *X configuration files*, e.g.
`/etc/X11/xorg.conf.d/00-keyboard.conf`
```
Section "InputClass"
    Identifier "system-keyboard"
    MatchIsKeyboard "on"
    Option "XkbModel" "pc104"
    Option "XkbLayout" "us,ru"
    Option "XkbVariant" ",mac"
    Option "XkbOptions" "grp:alt_shift_toggle"
EndSection
```
* with `localectl`, it will create *X configuration files* automatically:
`localectl set-x11-keymap us,ru pc105 ,mac grp:alt_shift_toggle`
it will also convert specified layout to closest match for console and applies to `vconsole.conf` (to avoid this, use `--no-convert`)

### frequently used XKB options
* `grp:.*toggle` keybinding for switch layout
* `terminate:.*` keybinding for terminate Xorg
* `ctrl:.* | caps:.*` various swaps and substitutions for *caps lock* and modifiers keys
* `keymap:pointerkeys` toggles mouse keys by `Shift+NumLock`
* `compose:.*` use existing key as *Compose* key: `<Compose> ' e` => `é`
  also custom *Compose* keys may be defined in `~/.XCompose`:
  ```
  include "%L"  # include system config
  <Multi_key> <g> <a> : "<symbol>"
  ```
* `lv3:lalt_switch` toggle 3rd layer, useful for various signs, e.g. `eurosign:5`

### one-click key functions
use `setxkbmap` to set `caps_lock` key to act as `Ctrl` key when pressed and hold as modifier
use `xcape -e 'Caps_Lock=Escape'` to set `caps_lock` to act as `Escape` key when it pressed once

## Xmodmap
`xmodmap` settings are reset by `setxkbmap` 
`xmodmap` settings are not applied to hotplug devices automatically

Types of keyboard values:
* `keycode` numeric representation received by the kernel
* `keysym` value assigned to `keycode`

### modify keymap
`xmodmap -pke` shows current keymap table
e.g. `keycode 57 = n N`
keysym column corresponds to following combinations:
- `Key`
- `Shift+Key`
- `Mode_switch+Key`
- `Mode_switch+Shift+Key`
- `ISO_Level3_Shift+Key`
- `ISO_Level3_Shift+Shift+Key`
to skip assigment `NoSymbol` can be used

`ISO_Level3_Shift` is `AltGr` on non-US keyboards

*NOTE*: all keysyms are available only for `us(intl)` layout

Special keysyms like `XF86AudioMute` are listed in `/usr/include/X11/XF86keysym.h`

Custom table can be created with `xmodmap -pke > ~/.Xmodmap` and following line in `xinitrc`:
`[[ -f ~/.Xmodmap ]] && xmodmap ~/.Xmodmap`
or temporary: `xmodmap -e "keycode 46 = l L l L lstroke Lstroke lstroke"`

### modifier keys
defaults:
- *mod1* `Alt_L`, `Meta_L`
- *mod2* `Num_Lock`
- *mod3* -
- *mod4* `Super_L`, `Super_R`, `Hyper_L`
- *mod5* `Level3_Shift`, `Mode_switch`

Additional modifier keys came from _Space Cadet Keyboard_
* `Meta` usually emulated with `AltGr` or `Alt_R` (sometimes it's called `Option`)
* `Super` is `Win` (Windows) or `Cmd` (MacOS)
* `Compose` is usually `Win Menu`
* `Meh` is `Shift + Ctrl + Alt`
* `Hyper` is gone, sometimes it's `Shift + Ctrl + Alt + Super`

`xmodmap -pm` shows current modifier keys table

Custom table can be created as described above
`~/.Xmodmap`
```
clear lock
clear mod2
keycode 38 = Caps_Lock
keycode 77 = Num_Lock
add lock = Caps_Lock
add mod2 = Num_Lock
```

### reverse scrolling
`~/.Xmodmap`
`pointer = 1 2 3 `*5* *4*` 7 6 8 9 10 11 12`

### swap mouse buttons
`~/.Xmodmap`
`pointer = 3 2 1`

### disable touchpad
`xinput disable $(xinput list | grep -Eio '(touchpad|glidepoint)\s*id\=[0-9]{1,2}' | grep -Eo '[0-9]{1,2}')`

# backlight
## mackbook
`/sys/class/leds/smc\:\:kbd_backlight/brightness`

# leds
* `setleds -L +num` - turns on NumLock Led (without changing VT flag)
