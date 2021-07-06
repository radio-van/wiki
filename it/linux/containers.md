# Contents

- [basics](#basics)
- [tools](#tools)

# basics
1. **container** - the container itself, that follows Open Container Initiative - **OCI** (multiple layers) or **LXD** (single layer)
2. tool for **spawning** containers, **OCI**-compliant: `runc` (Go), `crun` (C) - start and stop containers
3. **runtime** or **engine**: `containerd` (Docker), `CRI-O` (RedHat), `LXD`, `RKT`, `podman + libpod` - pulls images, do networking and storage
4. **control utility** for user: `docker` (Docker), `podman`, `kubectl` - create and run containers

*NOTE*: `kubernetes` interacts with **runtime** via **CRI** - Container Runtime Interface.  
`containerd` is compatible with **CRI** via plugin.

- `Docker` requires `dockerd` daemon running at root, with which `docker` utility talks to.  
- `podman` interacts with **spawning** tool directly, without a daemon

# tools
* [docker](../utilities/docker.md)
* [kubernetes](../utilities/kubernetes.md)
