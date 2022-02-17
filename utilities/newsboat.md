# Contents

- [reload w/o opening](#reload-wo-opening)
- [auto reload by cron](#auto-reload-by-cron)

# reload w/o opening
`newsboat -x reload`

# auto reload by cron
every 6 hours::
`* */6 * * * /usr/local/bin/newsboat -x reload`
:: see also [[wiki0:crontab|crontab]]
