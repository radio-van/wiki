# Contents

- [installation](#installation)
    - [alpine](#alpine)
- [troubleshooting](#troubleshooting)
    - [no host internet conection](#no-host-internet-conection)
    - [ESPHome can not find device](#esphome-can-not-find-device)
    - [reset password](#reset-password)
- [Recepies](#recepies)
    - [Tion error notification](#tion-error-notification)

# installation

## alpine
* `apk add --no-cache --virtual .build-deps binutils-gold g++ gcc gnupg libgcc linux-headers make libffi-dev openssl-dev python3-dev`
* `pip3 install homeassistant`
* `apk del .build-deps`


# troubleshooting

## no host internet conection
* chack `ha network info`
* set gateway as DNS server

## ESPHome can not find device
```yaml
zeroconf:
  ignore:
    - <entity>
```

OR

`<hass config>/.storage/core.config_entries`
check IPs in
```
domain:
  esphome:
```
and "real" ones (from routers)

## reset password
from local console:
`auth reset --username 'existing_user' --password 'new_password'`


# Recepies

## Tion error notification
```yaml
alias: Notify Problem
description: ""
triggers:
  - event_type: state_changed
    event_data: {}
    trigger: event
conditions:
  - condition: template
    value_template: >-
      {{  trigger.event.data.new_state is not none 

      and trigger.event.data.new_state.domain == 'binary_sensor' 

      and trigger.event.data.new_state.attributes.device_class | default('') ==
      'problem' 

      and trigger.event.data.new_state.state == 'on' 

      }}
actions:
  - wait_template: "{{ is_state(trigger.event.data.entity_id, 'on') }}"
    continue_on_timeout: true
    timeout: "00:00:10"
  - if:
      - condition: template
        value_template: "{{ not wait.completed }}"
    then:
      - metadata: {}
        data:
          message: >-
            ⚠️ Обнаружена проблема в {{
            device_attr(trigger.event.data.entity_id, "name") }}
            (`{{trigger.event.data.entity_id}}`)
        action: notify.me
mode: single
```

> Если нужна фиксированная скорость при старте, то, например, в on_boot можно климат принудительно переключить в этот режим
