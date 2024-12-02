# Contents

- [Installation](#installation)
- [flash firmware](#flash-firmware)

# Installation

* add user to `uucp` group
* load `cdc_acm` kernal module
* install corresponding board core, e.g. `arduino-cli core install arduino:avr`


# flash firmware
`arduino-cli upload -p /dev/tty... <sketch>`
