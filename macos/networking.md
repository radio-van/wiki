# Contents

- [enable ssh server](#enable ssh server)
- [NFS auto-mounting](#NFS auto-mounting)
- [set TTL](#set TTL)

# enable ssh server
`sudo systemsetup -setremotelogin on`

# NFS auto-mounting
`/etc/auto_nfs`
```
  /../Volumes/<volume>    -fstype=nfs,noowners,nolockd,noresvport,hard,bg,intr,rw,tcp,nfc nfs://<path>
```
`/etc/auto_master`

```
  #
  # Automounter master map
  #
  +auto_master            # Use directory service
  /net                    -hosts          -nobrowse,hidefromfinder,nosuid
  /home                   auto_home       -nobrowse,hidefromfinder
  /Network/Servers        -fstab
  /-                      -static
  /-                      auto_nfs        -nobrowse,nosuid
```

# set TTL
`sudo sysctl -w net.inet.ip.ttl=65`
