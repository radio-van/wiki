# Contents

- [filesystem](#filesystem)
- [apps](#apps)
    - [permissions](#permissions)
- [bootloader](#bootloader)
    - [slots](#slots)
- [encryption](#encryption)
- [building apk](#building-apk)
    - [create keystore](#create-keystore)
    - [sign](#sign)
    - [verify](#verify)
    - [zipalign](#zipalign)
- [flashing](#flashing)
    - [bootloader without slots](#bootloader-without-slots)
    - [bootloader with slots](#bootloader-with-slots)
    - [Galaxy 4](#galaxy-4)
- [Turn off location indicator](#turn-off-location-indicator)

# filesystem
- `/data` r/w data (configs, apps, userdata), wipes on reset
- `/data/app/com.<name>.<suffix>` apps
- `/data/data/com.<vendor>` place for storing apps data aka *internal storage*, access only with app's UID. This dir can be cleaned from settings app.
- `/system` base system, is mounted read-only, like `/usr` in Linux
- `/system/apps` pre-installed apps, can be overlayed by app in `/data`
- `/vendor` system data, specific for particular Android firmware, also like `/usr` in Linux
- `/sdcard` aka *external storage*, place for user data, like `/home` in Linux. Can be accessed by any app (with permission) or PC via USB. Real mountpoint is `/data/media/0`, also can be accessed at `/storage/emulated/0`
- `/sdcard/Android/data/com.<app>` storage for app's big data. Can be accessed w/o permission by any app. App's uninstallation *does not* clear this dir.

# apps
## permissions
Built-in apps (from `/system` partition) can use special permissions to reboot, delete other packages, record screen.
Such apps must be signed with the same key, as services they want access to, i.e. with system key of Android firmware image.

# bootloader
Android contains built-in *bootloader*.
It does:
- init Trust Environment
- find initramfs
- check integrity with sign
- load kernel, pass control to kernel

*recovery* is additional tiny OS to update/manage primary OS.
It can rewrite partitions by flashing them in special mode - *fastboot* (with *fastboot* protocol).
It will prevent downgrading Android system.

_unlocking_ bootloader means deactivating sign verifying for system and recovery. Wipes `/data` for security.
bootloader can be safely locked back *only* if boot, recovery and system are stock (have same version as bootloader).

## slots
`/boot`, `/system`, `/vendor` are duplicated (`a/b`).
`/data` is not.
One slot is *active* and used for boot.

# encryption
File-based aka per-file.
Most of `/data` is encrypted with key generated from user-password. If password is not set up at first launch, default is used.
Key is calculated each time system boots and not stored.

*device encrupted storage* is encrupted with key from Trusted Environment.
It is used for storing some app's data which must be accessible before user enters password (e.g. Alarms).

# building apk

## create keystore
```
   keytool -genkey -v -keystore mycustom.keystore -alias mykeyaliasname -keyalg RSA -keysize 2048 -validity 10000
 ```
## sign
```
   jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore mycustom.keystore -storepass mystorepass $APK.apk mykeyaliasname
 ```
## verify
```
   jarsigner -verify $APK.apk
 ```
## zipalign
```
   zipalign 4 $APK.apk $APK-signed.apk
 ```

# flashing


- activate Developer options (taps on build version)
- allow usb debugging
- allow OEM unlock
  
  several options are possible here, common one is:
    - boot to `fastboot mode` (usb + volume down)
    - unlock bootloader (warranty void!)
    ```
    fastboot flashing unlock
    ```
    - reboot phone

## bootloader without slots
* `fastboot flash recovery twrp-...img`
* `fastboot reboot`

## bootloader with slots

- get current boot slot, `a` or `b`
```
adb shell getprop ro.boot.slot_suffix
```
- or
```
fastboot getvar current_slot
```
- boot into `fastboot mode`, switch slot
```
fastboot --set-active=x
```
    where `x` is `a` or `b`
- flash *twrp img*
```
fastboot flash boot_ twrp-.img 
```
* `TWRP-SEP.img` is needed, stock *twrp* lack touch
- reboot to recovery from fastboot
- push *twrp-installer*
```
adb push twrp-installer-.zip /sdcard
```
- install *twrp-installer* from *twrp* -> now *TWRP* is installed to both slots
- reboot to opposite slot from *twrp*  # TODO not clear yet
- flash `Magisk`, `firmware`, etc

* seems like patched boot is needed to be flashed to get *Magisk* to work

## Galaxy 4

- boot to `downloading mode` (hold volume up + home + power)
```
  heimdall flash --RECOVERY twrp-.img --no-reboot
```
- reboot phone into recovery (volume down + power + home)
- flash `Magisk`, `firmware`, etc


# Turn off location indicator
in **adb shell**:
`cmd device_config put privacy location_indicators_enabled false default`
