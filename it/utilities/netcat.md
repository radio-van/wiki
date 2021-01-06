# Contents

- [simple web server](#simple-web-server)

# simple web server
```bash
while true ; do nc -l -p 1500 -c 'echo -e "HTTP/1.1 200 OK\n\n $(date)"'; done
```
or  
```bash
while true ; do nc -l -p 1500 -e /path/to/program ; done
```

