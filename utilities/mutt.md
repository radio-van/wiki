# Contents

- [flags](#flags)
- [selection](#selection)

# flags
* `+` message is to you and you only
* `T` message is to you, but also to or CC'ed to others
* `C` message is CC'ed to you
* `F` message is from you
* `L` message is sent to a subscribed mailing list

# selection
* `t` tags single message
* `T` tags messages matching pattern, with modifiers:
    * `.` all
    * `~m <range>` for matching numbers
    * `~b` body
    * `~B` whole message
    * `=b` body on IMAP server
    * `=B` whole message on IMAP server
    * `~d <range>` sent date in `<range>`
    * `~e` sender
    * `~D` deleted messages
    * ... [full list](http://www.mutt.org/doc/manual/manual.html#patterns)
* `;<action>` applies **action** (aka **delete**, **save**) to tagged messages
* `^t` untag messages, patterns are the same
