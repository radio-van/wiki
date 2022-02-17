# Contents

- [postscript](#postscript)

# postscript
If printer supports direct print (e.g. `jetdirect` on `9100` port of *HP* printer):  
`nc <IP> <port> < document.pdf`  
useful flags:  
- `-w <sec>` wait N sec before disconnection
- `-v` be verbous
If *PDF* is not supported, it can be converted to *PS*:  
`pdf2ps <source> <output>`
