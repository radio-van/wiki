# Contents

- [debug](#debug)
- [periodic tasks](#periodic tasks)

# debug
`--loglevel=DEBUG` is useful for investigation, e.g. it shows list of registered tasks.

# periodic tasks
`celery beat` is used to run tasks periodicaly.  

Schedule can be defined in `settings.py`:  

```python
CELERY_BEAT_SCHEDULE = {
    'task 1 name': {
        'task': '<task>',
        'schedule': datetime.timedelta(minutes=INTERVAL)
    },
    'task 2 name': {
        'task': '<task>',
        'schedule': crontab()  # every minute
    },
```
**NOTE**: `<task>` must be the full path to task function, i.e. `<app>.<folder>.<file>.<method>`  
