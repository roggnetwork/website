+++
title = 'Download'
weight = 2
+++

## Installation

### Pre-built Binaries

Download the platform binary from https://github.com/bgpgg-org/bgpgg/releases. Each release ships two binaries:

- `bgpggd` - the BGP daemon
- `ggsh` - the gg shell, used to manage `bgpggd`

### From Source

Requires Rust.

```bash
git clone https://github.com/bgpgg-org/bgpgg.git
cd bgpgg
make
```

Binaries land in `target/release/`.

### Docker

```bash
docker pull bgpgg/bgpgg
```

## Quick Start

Create `rogg.conf`:

```
service bgp {
  asn 65000
  router-id 1.1.1.1
  listen-addr 0.0.0.0:179

  peer 192.168.1.1 {
    remote-as 65001
  }
}
```

Start the daemon (defaults to `/etc/rogg/rogg.conf`):

```bash
./bgpggd --config rogg.conf
```

Use `ggsh` to query and manage the running daemon:

```bash
./ggsh show bgp summary
./ggsh show bgp routes
```

Or open an interactive session:

```bash
./ggsh
ggsh> show bgp summary
ggsh> exit
```

## Docker

Run with defaults:

```bash
docker run -d --name bgpggd -p 179:179 bgpgg/bgpgg
```

Use a config file:

```bash
docker run -d --name bgpggd -p 179:179 \
  -v $(pwd)/rogg.conf:/etc/rogg/rogg.conf \
  bgpgg/bgpgg
```

Use `ggsh` against the container:

```bash
docker exec bgpggd ggsh show bgp summary
docker exec bgpggd ggsh show bgp routes
```

For a multi-speaker example, see [`docker/docker-compose.yml`](https://github.com/bgpgg-org/bgpgg/blob/master/docker/docker-compose.yml).
