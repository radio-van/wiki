# Contents

- [add logs](#add-logs)
- [reset user password](#reset-user-password)
- [unlock app](#unlock-app)

# add logs
`/etc/syslog.conf`
e.g. `cron.*  /var/log/cron.log`

# reset user password
* hold `command-S` on startup.
* run `mount -uw /.`
* load *plist*:
  * *10.7 and later*:
    `launchctl load /System/Library/LaunchDaemons/com.apple.opendirectoryd.plist`
  * *10.6 and earlier*:
    `launchctl load /System/Library/LaunchDaemons/com.apple.DirectoryServices.plist`
* run `passwd username`
* run `reboot`

# unlock app
`sudo xattr -rd com.apple.quarantine /Applications/LockedApp.app`
