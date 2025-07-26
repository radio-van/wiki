# Contents

- [show progress](#show-progress)
- [clone over network](#clone-over-network)
- [clone over network into compressed file](#clone-over-network-into-compressed-file)

# show progress
`dd if=... of=... status=progress conv=fsync oflag=direct` 


# clone over network
`dd if=... | ssh user@server "dd of=..."`
NOTE: `bs` arg must be the same


# clone over network into compressed file
compression will be made on remote host side
`ssh <host> 'dd if=<dev> bs=... conv=sparse | xz -zce' > local_file.xz`
where:
- `conv=sparse` tries to seek forward rather than copy zeroes
- `-z` for compression
- `-c` for sending to stdout and keep original files
- `-e` for extreme compression
