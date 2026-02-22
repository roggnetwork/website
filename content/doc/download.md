+++
title = 'Download'
weight = 2
+++

## Installation

### Pre-built Binaries

Download the platform binary from https://github.com/bgpgg-org/bgpgg/releases

### From Source

Requires Rust

```bash
git clone https://github.com/bgpgg-org/bgpgg.git
cd bgpgg
make
```

### Docker

```bash
docker pull bgpgg/bgpgg
```

## Quick Start

Create `config.yaml`:

```yaml
asn: 65000
router_id: "1.1.1.1"
peers:
  - address: "192.168.1.1"
    asn: 65001
```

Start the daemon:

```bash
./bgpggd -c config.yaml
```

Use the CLI:

```bash
./bgpgg peer list
./bgpgg global rib show
```

## Docker

Run with defaults:

```bash
docker run -d --name bgpggd -p 1790:179 bgpgg/bgpgg
```

Configure via environment variables:

```bash
docker run -d --name bgpggd -p 1790:179 \
  -e BGPGG_ASN=65001 \
  -e BGPGG_ROUTER_ID=2.2.2.2 \
  bgpgg/bgpgg
```

Use a config file:

```bash
docker run -d --name bgpggd -p 1790:179 \
  -v $(pwd)/config.yaml:/config.yaml \
  bgpgg/bgpgg -c /config.yaml
```

Use the CLI:

```bash
docker exec bgpggd bgpgg peer list
docker exec bgpggd bgpgg global rib show
```
