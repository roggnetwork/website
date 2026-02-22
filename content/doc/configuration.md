+++
title = 'Configuration'
weight = 2
+++

## Configuration File

bgpgg uses YAML for configuration. Default location: `/etc/bgpgg/config.yaml`

Override with `-c` flag or `BGPGG_CONFIG_PATH` environment variable.

## Server Configuration

```yaml
asn: 65000                          # Required: Autonomous System Number
router_id: "1.1.1.1"               # Required: Router ID (IPv4 address)
listen_addr: "0.0.0.0:179"         # BGP listen address (default: 0.0.0.0:179)
grpc_listen_addr: "127.0.0.1:50051" # gRPC API address (default: 127.0.0.1:50051)
log_level: "info"                  # error, warn, info, debug, trace (default: info)
hold_time_secs: 180                # BGP hold time (default: 180)
connect_retry_secs: 30             # Connection retry interval (default: 30)
cluster-id: "1.1.1.1"              # Route reflector cluster ID (defaults to router_id)
sys_name: "bgpgg router"           # BMP system name (optional)
sys_descr: "BGP daemon"            # BMP system description (optional)
```

## Peer Configuration

```yaml
peers:
  - address: "192.168.1.1"              # Required: Peer IP address
    asn: 65001                          # Peer ASN (for validation)
    port: 179                           # BGP port (default: 179)
    passive_mode: false                 # Don't initiate connection (default: false)
    idle_hold_time_secs: 30             # Idle hold time (default: 30)
    damp_peer_oscillations: true        # Exponential backoff (default: true)
    max_prefix:
      limit: 1000
      action: "terminate"               # or "discard"
    graceful-restart:
      enabled: true                     # Default: true
      restart_time: 120                 # GR time in seconds (default: 120, max: 4095)
    rr-client: false                    # Route Reflector client
    add-path-send: "disabled"           # "disabled" (default) or "all"
    add-path-receive: false             # Accept multiple paths
    import-policy: []                   # List of import policy names
    export-policy: []                   # List of export policy names
```

## BMP Configuration

Monitor BGP with external collectors:

```yaml
bmp_servers:
  - address: "127.0.0.1:11019"
    statistics_timeout: 60              # Stats interval in seconds (0 to disable)
```

## Policy Configuration

### Defined Sets

Building blocks for matching:

```yaml
defined-sets:
  prefix-sets:
    - name: "internal-prefixes"
      prefixes:
        - prefix: "10.0.0.0/8"
          masklength_range: "16..24"    # or "exact"

  neighbor-sets:
    - name: "upstream-neighbors"
      neighbors:
        - "192.168.1.1"

  as-path-sets:
    - name: "upstream-asns"
      patterns:                          # Regex patterns
        - "^65001$"

  community-sets:
    - name: "no-export-communities"
      communities:
        - "65000:100"
        - "NO_EXPORT"
```

### Policy Definitions

```yaml
policy-definitions:
  - name: "import-policy"
    statements:
      - name: "accept-internal"
        conditions:
          match-prefix-set:
            set-name: "internal-prefixes"
            match-option: "any"          # or "all", "invert"
          match-neighbor-set:
            set-name: "upstream-neighbors"
            match-option: "any"
          route-type: "ebgp"             # or "ibgp", "local"
        actions:
          accept: true
          local-pref: 200
          community:
            operation: "add"              # or "remove", "replace"
            communities:
              - "65000:100"
```

### Match Options

- `any` - At least one element must match
- `all` - All elements must match
- `invert` - No elements must match

### Action Types

- `accept` / `reject` - Route decision
- `local-pref` - Local preference
- `med` - Multi-exit discriminator
- `community` / `ext-community` / `large-community` - Community manipulation

## Environment Variables

Override configuration file settings:

- `BGPGG_CONFIG_PATH` - Config file path
- `BGPGG_ASN` - Autonomous System Number
- `BGPGG_ROUTER_ID` - Router ID
- `BGPGG_LISTEN_ADDR` - BGP listen address
- `BGPGG_GRPC_LISTEN_ADDR` - gRPC listen address
- `BGPGG_LOG_LEVEL` - Log level
- `BGPGG_HOLD_TIME_SECS` - BGP hold time
- `BGPGG_CONNECT_RETRY_SECS` - Connection retry interval

## CLI Reference

### Peer Management

```bash
bgpgg peer list                         # List all peers
bgpgg peer show 192.168.1.1            # Show peer details
bgpgg peer add 192.168.1.1 65001       # Add peer
bgpgg peer del 192.168.1.1             # Delete peer
```

### Route Management

```bash
bgpgg global rib show                  # Show all routes
bgpgg global rib add 10.0.0.0/24 --nexthop 192.168.1.1
bgpgg global rib add 10.0.0.0/24 --nexthop 192.168.1.1 \
  --origin incomplete --as-path "100 200 300" \
  --local-pref 150 --med 50 --community 65001:100
bgpgg global rib del 10.0.0.0/24
```

### Server Information

```bash
bgpgg global info                      # ASN, router-id, uptime
bgpgg global summary                   # Statistics
```

### Custom gRPC Address

```bash
bgpgg --addr http://192.168.1.1:50051 peer list
```
