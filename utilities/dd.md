# Contents

- [show progress](#show-progress)
- [clone over network](#clone-over-network)


# show progress
`dd if=... of=... status=progress conv=fsync oflag=direct` 


# clone over network
`dd if=... | ssh user@server "dd of=..."`
NOTE: `bs` arg must be the same
