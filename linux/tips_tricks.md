# Contents

- [fork bomb](#fork-bomb)
- [Debian](#debian)
    - [apt proxy](#apt-proxy)
    - [autologin in lightdm](#autologin-in-lightdm)
    - [locale](#locale)
    - [keyboard switch](#keyboard-switch)
- [share terminal](#share-terminal)
    - [get output from process executed by another thread](#get-output-from-process-executed-by-another-thread)
    - [version of distro](#version-of-distro)
- [QR code for WiFi](#qr-code-for-wifi)

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

## apt proxy
`/etc/apt/apt.conf.d/...` or `/etc/apt/apt.conf`  
```
Acquire::http::Proxy "http://username:password@proxy.server:port/";
```

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

## version of distro
`cat /etc/os-release`


# QR code for WiFi
`qrencode "WIFI:T:WPA;S:My_Network;P:My_very_secure_Password;;" -o wifi_login.png`
