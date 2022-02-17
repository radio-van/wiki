# Contents

- [token from rooted device](#token from rooted device)
- [manual wi-fi configuration via ssh](#manual wi-fi configuration via ssh)

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
