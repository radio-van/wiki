# Contents

- [Build](#build)
    - [UnionFS](#unionfs)
    - [Base image](#base-image)
    - [Dockerfile](#dockerfile)
    - [transfer images](#transfer-images)
    - [ONBUILD](#onbuild)
    - [multistage build](#multistage-build)
    - [run as non-root user](#run-as-non-root-user)
    - [resources constraints](#resources-constraints)
- [Containers](#containers)
    - [recreate container](#recreate-container)
    - [removing container](#removing-container)
    - [stop all containers](#stop-all-containers)
    - [rm all containers](#rm-all-containers)
- [Networks](#networks)
    - [workaround existing endpoint](#workaround-existing-endpoint)
- [Installation](#installation)
    - [Debian](#debian)
- [Managing](#managing)
    - [clear logs](#clear-logs)
    - [cleanup](#cleanup)
- [Configuration](#configuration)
    - [runtime](#runtime)
    - [data location](#data-location)
        - [transfer to new location](#transfer-to-new-location)
- [Usage](#usage)
    - [Dockerfile](#dockerfile-2)
    - [CMD and ENTRYPOINT](#cmd-and-entrypoint)
- [Tools](#tools)
    - [docker-compose](#docker-compose)
        - [healthcheck](#healthcheck)
        - [run only when other service is healthy](#run-only-when-other-service-is-healthy)
        - [set ARG from .env](#set-arg-from-env)
        - [color in logs](#color-in-logs)
- [Podman](#podman)
    - [replace Docker with podman](#replace-docker-with-podman)
    - [auth](#auth)
    - [autostart podman-compose](#autostart-podman-compose)
    - [troubleshootig](#troubleshootig)
        - [network driver errors](#network-driver-errors)
        - [missing dynamic library](#missing-dynamic-library)

# Build

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
* to copy dir rather than its content use `ADD <src> <dest>` instead of `COPY`  

## transfer images
* `docker save <image> > <image_file>.tar`
* `docker load < <image_file>.tar>`
OR
`docker save <image> | ssh -C <host> docker load`


## ONBUILD
`ONBUILD <command>` allows to run `<command>` during building _another_ image based on this one
**NOTE**: `podman` doesn't support `ONBUILD` unless `--format docker` is provided

## multistage build
```bash
# Build stage
FROM python:3.10-slim AS builder

WORKDIR /app
COPY app.py .
RUN pip install --target=/tmp/deps psycopg2-binary

# Final stage
FROM python:3.10-slim

WORKDIR /app
COPY --from=builder /app/app.py .
COPY --from=builder /tmp/deps /usr/local/lib/python3.10/site-packages/

CMD ["python", "app.py"]
```

## run as non-root user
```bash
RUN useradd -m etluser
USER etluser
```

## resources constraints
```yaml
app:
  build: .
    mem_limit: 512m
      cpu_shares: 256
```


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


# Networks

## workaround existing endpoint
Caused by attached container that doesn't exists.

* check `docker network inspect --format '{{range $cid,$v := .Containers}}{{printf "%s: %s\n" $cid $v.Name}}{{end}}' "<network>"`
to find out problematic container
* `docker disconnect -f <network> <container>`


# Installation

## Debian

* add source
    ```
    /etc/apt/sources.list.d/10-debian.list

    deb [arch=amd64 signed-by=/usr/share/keyrings/docker-keyring.gpg] https://download.docker.com/linux/debian bookworm stable
    ```
* add signature
    ```
    /usr/share/keyrings/docker-keyring.gpg
    ```

    Output of `wget -O- https://download.docker.com/linux/debian/gpg | gpg --dearmor`
    `gpg --dearmor` is necessary if `file <gpg.gpg>` outputs `docker-keyring.gpg: PGP public key block Public-Key (old)`
* install
    `apt update && apt install docker-ce`


# Managing

## clear logs
`echo "" > $(docker inspect --format='{{.LogPath}}' <container_name_or_id>)`

## cleanup
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


# Usage

## Dockerfile

## CMD and ENTRYPOINT
* `ENTRYPOINT` can be overrided **only** by explicitly passing `--entrypoint` param on container run.
  args that are passed on container run are appended to `ENTRYPOINT` command
* `CMD` defines default arguments for container run and is overrided by user deefined args on container run

So, `entrypoint` defines the command and `CMD` defines default args for that command


# Tools

## docker-compose

### healthcheck
```yaml
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 2s
      retries: 5
```

### run only when other service is healthy
```yaml
  app:
    build: .
    depends_on:
      db:
        condition: service_healthy
```


### set ARG from .env
`FROM` derective resets environment, so `ARG` should be declared after, except for image version.

`ARG` are used in `Dockerfile` and used only during *build*  
`.env` file is used only by `docker-compose`  

```
build:
  args:
    ARG1: value
    ARG2: value
  context: .
```

### color in logs
add `tty: true` to `docker-compose.yaml`


# Podman

## replace Docker with podman
* `touch /etc/subuid /etc/subgid`
* `usermod --add-subuids 100000-165535 --add-subgids 100000-165535 <username>`
* `systemctl --user start podman.service`
* since `4.0` **netavark** becomes the default instead of **CNI**, additional packages are required:
  * `netavark`
  * `aardvark-dns`
  * set `network_backend = "netavark"` in `/etc/containers/containers.conf`
* `export DOCKER_HOST="unix://$XDG_RUNTIME_DIR/podman/podman.sock"`
* edit `/etc/containers/registries.conf`:
    * `unqualified-search-registries = ['docker.io']`

## auth
move `.docker/auth.json` to `$XDG_RUNTIME_DIR/containers/`

e.g. of `auth.json` for _yandex cloud_:
```
{
  "auths": {
    "cr.yandex": {
      "auth": "<token>"
    }
  }
}
```

OR

`/run/user/1000/containers/auth.json`
```json
{
  "credHelpers": {
    "cr.yandex": "yc"
  }
}
```

## autostart podman-compose
* `podman-compose systemd -a create-unit`
* `podman-compose -f <docker-compose-name>.yaml systemd -a register`
* `systemctl --user enable --now podman-compose@<name>`

NOTE: `sudo loginctl enable-linger '<username>'` might be necessary


## troubleshootig

### network driver errors

* **Error bad parameter: Link is not supported** - `links:` in docker-compose is a deprecated option
* **crun: setrlimit RLIMIT_NPROC: Operation not permitted: OCI permission denied** - use `--force-recreate` with compose command
* **failed to mount runtime directory for rootless netns: no such file or directory** - use proper net package?
* 
    ```
    running `/usr/bin/newuidmap 16111 0 500 1 1 10000 65536`: newuidmap: open of uid_map failed: Permission denied 
    Error: cannot set up namespace using "/usr/bin/newuidmap": exit status 1
    ```
    check capabilities:
    ```
    getcap /usr/bin/newuidmap /usr/bin/newgidmap
        /usr/bin/newuidmap cap_setuid=ep
        /usr/bin/newgidmap cap_setgid=ep
    ```
    if not, set:
    ```
    sudo chmod u+s /usr/bin/newuidmap
    sudo chmod u+s /usr/bin/newgidmap
    ```
    
### missing dynamic library

* error `msg: exec container process (missing dynamic library?) /usr/local/bin/docker-entrypoint.sh: No such file or directory...`
  remove image (or all images `docker images | awk '{print $3}' | xargs -I% docker rmi -f %`) and re-download them
