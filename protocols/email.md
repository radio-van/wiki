# Contents

- [terminology](#terminology)
- [address](#address)
- [protocols](#protocols)
- [systems](#systems)
- [encryption](#encryption)

# terminology
- `MUA` *Mail User Agent* stand-alone program or web-interface to intercat with emails
- `MSA` *Mail Submission Agent* receives message from *MUA*, determines destination via *DNS* and sends mail to *MTA*. Usually has address like `smtp.<domain>.<zone>`
- `MX` *Mail Exchange Server* lists available *MTAs* on this server. Usually has address `mx.<domain>.<zone>`
- `MTA` *Mail Transfer Agent* transfers emails between *PES* / *WES* / other *MTA*
- `MRA` *Mail Retrieval Agent* fetches mail from servers and send them to *MDA*
- `MDA` *Message Delivery Agent* stores messages until they are accessed by *MUA*

- `Open Mail Relays` receives and forwards messages to any recepient. Deprecated.

# address
- `@<domain>` is address of destination *MTA*
- *MUA* attachs a header with sender's *MTA*
- all intermediate *MTA* will add their own headers, they are omitted by *MUA*. Sender's address (aka FROM:) can be faked, but *MTA* headers can not

# protocols
- `SMTP` *Simple Mail Transfer Protocol* is used to transfer mail between *MUA* / *MTA*. Port `25 / 587 / 465 (SSL)`
- `POP3` *Post Office Protocol* downloads and deletes mail from server to client. Port `110 / 995 (SSL)`
- `IMAP` *Internet Message Access Protocol* allows to interact with mailbox from multiple devices. Port `143 / 993 (SSL)`
- `MIME` adds header to handle non-plain-text body of the message
- `MX` record listing *Mail Exchange Servers* on this domain

# systems
* *private email system* is accessed by stand-alone application, can be accessed only within private network
* *web based email system* stored on web-servers, can be accessed from any PC with internet connection

# encryption
- `S/MIME` based on certificates
- `PGP` based on web-of-trust

- `STARTTLS` is a `TLS/SSL` security layer to add encryption to communicating between mail servers. Does not prevent message to be read on intermediate servers
