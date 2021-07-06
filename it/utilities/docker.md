# Contents

- [build](#build)
    - [UnionFS](#build#UnionFS)
    - [Base image](#build#Base image)
    - [Dockerfile](#build#Dockerfile)
- [Containers](#Containers)
    - [recreate container](#Containers#recreate container)
    - [removing container](#Containers#removing container)
    - [stop all containers](#Containers#stop all containers)
    - [rm all containers](#Containers#rm all containers)
- [Managing](#Managing)
    - [unused stuff](#Managing#unused stuff)
- [Configuration](#Configuration)
    - [runtime](#Configuration#runtime)
    - [data location](#Configuration#data location)
        - [transfer to new location](#Configuration#data location#transfer to new location)

# build
## UnionFS
Docker uses *UnionFS* which represents all image layers as one read-only layer and combines it w/ read-write layer (to make container's fs writable).<br>
Due to *UnionFS* nature, if file which is present on bottom layer is deleted on above layer, it still will be present in container fs.<br>
To avoid this, use combined (`&&`) commands in `RUN` derective to create only one layer or use `--squash` option for `docker build` command to merge all layers.<br>
Efficiency of container could be checked with `dive` utility.

## Base image
- *Alpine* ~6MB
- *Debian stretch slim* ~55MB
- *Ubuntu* ~73MB
- *openSUSE* ~102MB
- *CentOS* ~204MB

Docker image size could be reduced with `DockerSlim` utility

## Dockerfile
* to avoid waste of space, clear caches, e.g:
```
    APK: ... && rm -rf /etc/apk/cache
    YUM: ... && rm -rf /var/cache/yum
    APT: ... && rm -rf /var/cache/apt
```
* do not use `chown` command in separate `RUN` derective (it will cause doubling folder size),<br>
  use `COPY --chown <user>` instead
  
# Containers
## recreate container
`docker-compose up <container>`
`--force-recreate` - restarts container even if configuration unchanged
`-V` recreates anonymous volumes

## removing container
`docker rm <container>`
`docker-compose down <container>`
`-v` removes all named/anonymous volumes

## stop all containers
`docker stop $(docker ps -a -q)`
## rm all containers
`docker rm $(docker ps -a -q)`

# Managing
## unused stuff
* check occupied space
`docker system df`, last column will show how much space could be free
* clear
`docker system prune` to remove all unused images, volumes, stooped containers and build caches
to remove selectively, use:
  * `docker container prune` to remove all stopped containers
  * `docker image prune` to remove `dangling` images (aka intermediate unused imeges),<br>
    could be previewed by `docker image ls -f dangling=true`
  * `docker volume prune` remove unused volumes
  * `docker build prune` remove unused cache

# Configuration
## runtime
`/etc/docker/daemon.json`
```
{
  "default-runtime": "runc",
  "runtimes": {
    "kata-runtime": {
      "path": "/opt/kata/bin/kata-runtime",
      "runtimeArgs": [
              "--kata-config /etc/kata/configuration.toml"
      ]
    },
    "crun": {
      "path": "/usr/local/bin/crun"
    }
  }
}
```
## data location
`/etc/docker/daemon.json`
```
{
  "data-root": "<path>"
}
```
OR  
`/lib/systemd/system/docker.service`
`ExecStart=/usr/bin/dockerd -g <new/path> -H fd://`

### transfer to new location
* `systemctl stop docker`
* *edit config*
* `systemctl daemon-reload`
* `rsync -aqxP /var/lib/docker/ /new/path/docker`
* `systemctl start docker`
