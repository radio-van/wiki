# Contents

- [installation](#installation)
    - [alpine](#alpine)

# installation
## alpine
* `apk add --no-cache --virtual .build-deps binutils-gold g++ gcc gnupg libgcc linux-headers make libffi-dev openssl-dev python3-dev`
* `pip3 install homeassistant`
* `apk del .build-deps`
