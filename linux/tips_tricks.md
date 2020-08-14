## Contents =
    - [[#fork bomb|fork bomb]]
    - [[#Debian|Debian]]
        - [[#Debian#autologin in lightdm|autologin in lightdm]]
        - [[#Debian#locale|locale]]
        - [[#Debian#keyboard switch|keyboard switch]]

## fork bomb =
{{{
   :(){ :|:& };:
}}}
explanation::
{{{
   :(){
    :|:&
   };:
}}}
:: aka
{{{
   function(){
     call_itself | call_itself &[background]
   };(end) call_function
}}}

## Debian =
### autologin in lightdm ==
`/usr/share/lightdm/lightdm.conf.d/01_debian.conf`

{{{
    [SeatDefaults]
    autologin-user=<username>
    autologin-user-timeout=0
}}}

### locale ==
`sudo dpkg-reconfigure locales`

`sudo apt-get install console-cyrillic`
`sudo dpkg-reconfigure console-cyrillic`

### keyboard switch ==
`/etc/default/keyboard`
`dpkg-reconfigure keyboard-configuration`


