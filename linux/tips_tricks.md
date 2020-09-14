# Contents

- [fork bomb](#fork-bomb)
- [Debian](#debian)
    - [autologin in lightdm](#autologin-in-lightdm)
    - [locale](#locale)
    - [keyboard switch](#keyboard-switch)
- [share terminal](#share-terminal)

# fork bomb
```
   :(){ :|:& };:
```
explanation
```
   :(){
    :|:&
   };:
```

aka   

```
   function(){
     call_itself | call_itself &[background]
   };(end) call_function
```

# Debian
## autologin in lightdm
`/usr/share/lightdm/lightdm.conf.d/01_debian.conf`

```
    [SeatDefaults]
    autologin-user=<username>
    autologin-user-timeout=0
```

## locale
`sudo dpkg-reconfigure locales`

`sudo apt-get install console-cyrillic`
`sudo dpkg-reconfigure console-cyrillic`

## keyboard switch
`/etc/default/keyboard`
`dpkg-reconfigure keyboard-configuration`

# share terminal
`<command> | nc seashells.io 1337`
