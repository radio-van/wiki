# Contents

- [security](#security)
    - [DNSCrypt](#dnscrypt)
    - [DNS-over-TLS](#dns-over-tls)
    - [DNS=over-HTTPS](#dnsover-https)
    - [DNSSEC](#dnssec)

# security
## DNSCrypt
Encrypts only sensitive information, like domain and IP.
Still clearly visible:
- that it is your query
- which DNS server answers

Also it is not well-supported.

## DNS-over-TLS
Establishes secure TLS connection. All DNS queries are made through this tunnel.
Standart.

## DNS=over-HTTPS
Some kind of hack, that mimics DNS queries to HTTPS traffic.
Not a standart.

## DNSSEC
Only authenticates DNS server, all DNS answers are signed.

