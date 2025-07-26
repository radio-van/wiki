# Contents

- [Installation](#installation)
- [flash firmware](#flash-firmware)
- [add ESP8266](#add-esp8266)

# Installation

* add user to `uucp` group
* load `cdc_acm` kernal module
* install corresponding board core, e.g. `arduino-cli core install arduino:avr`


# flash firmware
`arduino-cli upload -p /dev/tty... <sketch>`


# add ESP8266
* `arduino-cli config add board_manager.additional_urls https://arduino.esp8266.com/stable/package_esp8266com_index.json`
* `arduino-cli core update-index`
* `arduino-cli core install esp8266:esp8266`
