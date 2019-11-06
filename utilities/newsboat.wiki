= Contents =
    - [[#usage|usage]]
        - [[#usage#reload w/o opening|reload w/o opening]]
    - [[#tips&tricks|tips&tricks]]
        - [[#tips&tricks#auto reload by cron|auto reload by cron]]

= usage =
== reload w/o opening ==
`newsboat -x reload`

= tips&tricks =
== auto reload by cron ==
every 6 hours::
`* */6 * * * /usr/local/bin/newsboat -x reload`
:: see also [[wiki0:crontab|crontab]]
