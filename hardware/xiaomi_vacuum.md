[#](#) Contents

- [token from rooted device](#token from rooted device)
- [manual wi-fi configuration via ssh](#manual wi-fi configuration via ssh)
- [S6](#S6)

# token from rooted device
`printf $(cat /mnt/data/miio/device.token) | xxd -p`


# manual wi-fi configuration via ssh
```bash
echo ssid=\"WLAN_SSID\" > /mnt/data/miio/wifi.conf
echo psk=\"password\" >> /mnt/data/miio/wifi.conf
echo key_mgmt="WPA" >> /mnt/data/miio/wifi.conf
echo uid=1234567890 >> /mnt/data/miio/wifi.conf
echo 1234567890 > /mnt/data/miio/device.uid
echo "" > /mnt/data/miio/device.country
```

# S6

S6/T6 Rooting Cheatsheet
28.06.2020 Dennis Giese (https://dontvacuum.me/)

If you find issues with this document or have improvements, please contribute here: https://github.com/dgiese/dustbuilder-howto/

If you have a Roborock robot, these channels might be relevant for you:
- Dust announce channel (channel for firmware/Valetudo updates and info): https://t.me/dust_announce
- Roborock S5/S6 user group (for people that have Roborock robots) https://t.me/+CvzALOw70oM2MzUy
- Valetudo user group (for Valetudo users, not only Roborock robots) https://t.me/+NTCnSnuQYTA2ZTgy

Depending on the production date of your robot, the "vinda" file might not exist. Use Method 2) in that case


1) Try first if robot is manufactured before 06/2020

a) Do a full factory reset (keep "Home" button pressed + press reset button)
b) Connect over serial, as shown in the video

In U-Boot
----------

```
ext4load mmc 2:6 40008000 vinda
md 40008000
run setargs_mmc boot_normal
```

Hint: If the vinda step fails, use the second method below!

In Linux
--------

```
iptables -F
(you can run the next commands over SSH)
sed -i -e '/    iptables -I INPUT -j DROP -p tcp --dport 22/s/^/#/g' /opt/rockrobo/watchdog/rrwatchdoge.conf
sed -i -E 's/dport 22/dport 29/g' /opt/rockrobo/watchdog/WatchDoge
sed -i -E 's/dport 22/dport 29/g' /opt/rockrobo/rrlog/rrlogd
```
```
mkdir /mnt/recovery
mount /dev/mmcblk0p7 /mnt/recovery
sed -i -e '/    iptables -I INPUT -j DROP -p tcp --dport 22/s/^/#/g' /mnt/recovery/opt/rockrobo/watchdog/rrwatchdoge.conf
sed -i -E 's/dport 22/dport 29/g' /mnt/recovery/opt/rockrobo/watchdog/WatchDoge
sed -i -E 's/dport 22/dport 29/g' /mnt/recovery/opt/rockrobo/rrlog/rrlogd
umount /mnt/recovery
```


2) Try this when your robot is manufactured after 06/2020

a) Do a full factory reset (keep "Home" button pressed + press reset button)
b) Connect over serial, as shown in the video

In U-Boot
----------
```
setenv boot_fs a
setenv setargs_mmc ${setargs_mmc} init=/bin/sh
boot
```

In Linux
--------
As soon as the system boots (you have 5 seconds):
```
echo 'V' > /dev/watchdog

sed -i -e '/    iptables -I INPUT -j DROP -p tcp --dport 22/s/^/#/g' /opt/rockrobo/watchdog/rrwatchdoge.conf
sed -i -E 's/dport 22/dport 29/g' /opt/rockrobo/watchdog/WatchDoge
sed -i -E 's/dport 22/dport 29/g' /opt/rockrobo/rrlog/rrlogd

chown root:root /root
chmod 700 /root
mkdir /root/.ssh
chmod 700 /root/.ssh
```

Then write your pubkey into /root/.ssh/authorized_keys (make sure file has 600 permission)
you can use this (but you should generate your own!):
```
echo "ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAtzP7KEOiFtudnn8ks5jYlRqmu8yIgzvC+VH6VReH/s5zUIIdKUTxEKWnKX+1O5l6PGNj20/Szs/uHhQdV8658th+8rP5pP0zjELdtq1UTSdOU9Xs9aCjY/9NUbD5Sy5PE+vIU6ECpAx6hrs+Bh0C4jpIyHo2bA/OPISl1qvF/aVpe97Ae6ZWxbc6lBUTcH/WyxjQRtzK0DFEGfyyqyc/L3BXkNAeF9wir3ID1nruU69f1nxkH3SeB3Kdgb2G7Q5/jtUoRZZ3iy/aVNL48Wlb/Mb8kLSxgmCcPlO46F1WJVUiiIoR0SdPI1jj7ndmKzmViqLgOX32OwRSOndlX/GeCw== rsa-key-for-initial-root" > /root/.ssh/authorized_keys
```

You can find the key here:
https://builder.dontvacuum.me/initial-root-key.ppk
https://builder.dontvacuum.me/initial-root-key.id_rsa
