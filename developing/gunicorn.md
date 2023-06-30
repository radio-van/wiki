# Contents

- [basics](#basics)
- [gunicorn](#gunicorn)
    - [options](#options)
- [uvicorn](#uvicorn)
    - [options](#options-2)

# basics
    In production, built-in [Django](./django.md) webserver is not a good solution.  
    Generally, **WSGI** server is used as a middleware between Django and Apache/Nginx.  
    E.g. `Gunicorn`. Technically **WSGI** is a web-server by itself, but it cannot, for example  
    serve static files, thats why other webserver is placed in front of it.


# gunicorn

## options
* `--workers` determines number of concurrent requests
* `--bind=<address:port>` listen to on given address:port


# uvicorn

## options
* `--host`, `--port`
* `--workers` determines number of concurrent requests
* `<project_name>.asgi:application` <APP> argument
