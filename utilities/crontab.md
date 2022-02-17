# Contents

- [syntax](#syntax)
- [configuration](#configuration)
- [examples](#examples)
    - [newsboat crontab](#newsboat-crontab)
    - [git fetch crontab](#git-fetch-crontab)

# syntax
```
* * * * *  command to execute
│ │ │ │ │
│ │ │ │ └─── day of week (0 - 6) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0)
│ │ │ └──────── month (1 - 12)
│ │ └───────────── day of month (1 - 31)
│ └────────────────── hour (0 - 23)
└─────────────────────── min (0 - 59)
```

# configuration
to add logs on MacOS see [add logs](../macos/system.md#add-logs)

# examples
## newsboat crontab
see [newsboat](newsboat.md)

## git fetch crontab
`* * * * * su -s /bin/sh nobody -c 'cd <path> && /usr/local/bin/git pull -q origin master'`
for MacOS `* * * * * /bin/sh -c 'cd <path> && /usr/local/bin/git pull -q origin master'`
