# Contents

- [send via netcat](#send-via-netcat)
- [listen on port](#listen-on-port)
- [simple web server](#simple-web-server)
    - [output command](#output-command)
- [one-time file server](#one-time-file-server)
- [simple proxy](#simple-proxy)

# send via netcat
* TCP `echo 'TEXT' | nc <IP> <PORT>`
* UPD `echo 'TEXT'| nc -u <IP> <PORT>`


# listen on port
* TCP `netcat -l -p <PORT>`
* UDP `netcat -l -u -p <PORT>`


# simple web server

## output command
```bash
while true ; do nc -l -p 1500 -c 'echo -e "HTTP/1.1 200 OK\n\n $(date)"'; done
```
or  
```bash
while true ; do nc -l -p 1500 -e /path/to/program ; done
```

# one-time file server
```bash
{ printf 'HTTP/1.0 200 OK\r\nContent-Length: %d\r\n\r\n' "$(wc -c < <file>)"; cat <file> } | nc -l <port>
```

# simple proxy
using named pipe:  
```bash
mkfifo backpipe
nc -l 12345 0<backpipe | nc www.google.com 80 1>backpipe
```

or one-time proxy:  
```bash
ncat -l 12345 -c 'nc www.google.com 80'
```
