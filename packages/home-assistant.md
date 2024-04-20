# Contents

- [installation](#installation)
    - [alpine](#alpine)
- [troubleshooting](#troubleshooting)
    - [no host internet conection](#no-host-internet-conection)

# installation

## alpine
* `apk add --no-cache --virtual .build-deps binutils-gold g++ gcc gnupg libgcc linux-headers make libffi-dev openssl-dev python3-dev`
* `pip3 install homeassistant`
* `apk del .build-deps`


# troubleshooting

## no host internet conection
* chack `ha network info`
* set gateway as DNS server
