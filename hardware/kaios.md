# Contents

- [Nokia 8110](#Nokia 8110)
    - [flash with adb](#Nokia 8110#flash with adb)
    - [flash with EDL](#Nokia 8110#flash with EDL)
    - [install Wallace](#Nokia 8110#install Wallace)

# Nokia 8110

## flash with adb
* obtain **root** with [wallace](#install Wallace) or with *GerdaRecovery*
* push image to `/sdcard`
* `adb shell`
* `dd if=/sdcard/<image.img> of=/dev/block/bootdevice/by-name/<partition> bs=2048`

## flash with EDL
* clone `edl.py` for 8110: [repo](git@github.com:andybalholm/edl.git)
* prepare tools (venv is okay):  
  ```
  pip install pyusb pyserial capstone keystone-engine
  ```
  additionally (not tested, probably not required):
  ```
  append blacklist qcserial to /etc/modprobe.d/blacklist.conf
  copy 51-edl.rules to /etc/udev/rules.d
  copy 50-android.rules to /etc/udev/rules.d
  ```
* enter EDL mode:  
  * hold `up` and `down`
  * press `power`
  * screen should blink and stay black
  * attach cable
* `python3 edl.py -loader 8110.mbn -w <partition> <partition.img>`

## install Wallace
* activate **debug mode**: `*#*#33284#*#*`
* cd into source of `gdeploy` from [this repo](https://gitlab.com/suborg/gdeploy)
* `npm i && npm link`
* `gdeploy install <wallace dir>`
* additionally install: *GerdaFileManager*
