# Contents

- [fork bomb](#fork bomb)
- [Debian](#Debian)
    - [autologin in lightdm](#Debian#autologin in lightdm)
    - [locale](#Debian#locale)
    - [keyboard switch](#Debian#keyboard switch)
- [share terminal](#share terminal)
    - [get output from process executed by another thread](#share terminal#get output from process executed by another thread)

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

## get output from process executed by another thread
`strace -p<PID> -s9999 -e write`  
- `write` means capture output
- `-s9999` avoids truncating to 32 chars
