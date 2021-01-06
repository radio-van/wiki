# Contents

- [configuration](#configuration)
- [markup syntax](#markup-syntax)
- [shortcuts](#shortcuts)

# configuration
example `/usr/share/dunst/dunstrc`
per-user `$HOME/.config/dunst/dunstrc`

# markup syntax
```bash
    %a  appname
    %s  summary
    %b  body
    %i  iconname (including its path)
    %I  iconname (without its path)
    %p  progress value if set ([  0%] to [100%]) or nothing
```

# shortcuts
* `C-space` close notification
* `C-Shift-space` close all notifications
* `C-grave` show history
* `C-Shift-period` context menu
