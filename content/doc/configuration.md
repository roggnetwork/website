+++
title = 'Configuration'
weight = 3
+++

## Configuration File

bgpgg uses YAML for configuration. Default location: `/etc/bgpgg/config.yaml`

Override with `-c` flag or `BGPGG_CONFIG_PATH` environment variable:

```bash
# Using -c flag
bgpggd -c /path/to/custom-config.yaml

# Using environment variable
BGPGG_CONFIG_PATH=/path/to/custom-config.yaml bgpggd
```

## Server Configuration

```yaml
asn: 65000                            # Required: Autonomous System Number
router-id: "1.1.1.1"                  # Required: Router ID (IPv4 address)
listen-addr: "0.0.0.0:179"            # Optional: BGP listen address (default: 0.0.0.0:179)
grpc-listen-addr: "127.0.0.1:50051"   # Optional: gRPC API address (default: 127.0.0.1:50051)
log-level: "info"                     # Optional: Log level (default: info)
hold-time-secs: 180                   # Optional: BGP hold time (default: 180)
connect-retry-secs: 30                # Optional: Connection retry interval (default: 30)
cluster-id: "1.1.1.1"                 # Optional: Route reflector cluster ID (defaults to router-id)
sys-name: "bgpgg router"              # Optional: BMP system name
sys-descr: "BGP daemon"               # Optional: BMP system description
```

### log_level

- `error` - Only errors
- `warn` - Warnings and errors
- `info` - Informational messages, warnings, and errors
- `debug` - Debug messages and above
- `trace` - All messages including trace-level details

### Environment Variables

Server configuration can be overridden using environment variables:

- `BGPGG_CONFIG_PATH` - Config file path
- `BGPGG_ASN` - Autonomous System Number
- `BGPGG_ROUTER_ID` - Router ID (`router-id`)
- `BGPGG_LISTEN_ADDR` - BGP listen address (`listen-addr`)
- `BGPGG_GRPC_LISTEN_ADDR` - gRPC listen address (`grpc-listen-addr`)
- `BGPGG_LOG_LEVEL` - Log level (`log-level`)
- `BGPGG_HOLD_TIME_SECS` - BGP hold time (`hold-time-secs`)
- `BGPGG_CONNECT_RETRY_SECS` - Connection retry interval (`connect-retry-secs`)

## Peer Configuration

Optional. Define BGP peers:

```yaml
peers:
  - address: "192.168.1.1"          # Required: Peer IP address
    asn: 65001                      # Optional: Peer ASN (for validation)
    port: 179                       # Optional: BGP port (default: 179)
    passive-mode: false             # Optional: Don't initiate connection (default: false)
    idle-hold-time-secs: 30         # Optional: Idle hold time (default: 30)
    damp-peer-oscillations: true    # Optional: Exponential backoff (default: true)
    max-prefix:                     # Optional: Prefix limit
      limit: 1000
      action: "terminate"
    graceful-restart:               # Optional: GR settings
      enabled: true                 # Default: true
      restart-time: 120             # GR time in seconds (default: 120, max: 4095)
    rr-client: false                # Optional: Route Reflector client (default: false)
    add-path-send: "disabled"       # Optional: Add-path send mode (default: disabled)
    add-path-receive: false         # Optional: Accept multiple paths (default: false)
    import-policy: []               # Optional: List of import policy names
    export-policy: []               # Optional: List of export policy names
    rs-client: false                # Optional: Route server client mode RFC 7947 (default: false)
    enforce-first-as: true          # Optional: Enforce first AS in AS_PATH matches peer ASN (default: true)
    md5-key-file: ""                # Optional: Path to TCP MD5 key file RFC 2385 (chmod 600)
    next-hop-self: false            # Optional: Rewrite NEXT_HOP to local address when advertising (default: false)
```

### max_prefix

- `terminate` - Drop the BGP session when limit is exceeded
- `discard` - Silently ignore prefixes beyond the limit, keep session up

### add-path-send

- `disabled` - Send only the best path (default)
- `all` - Advertise all available paths to the peer

### rs-client

Mark this peer as a route server client (RFC 7947). In route server mode, bgpgg forwards routes between clients without modifying AS_PATH or NEXT_HOP (transparency mode).

Constraints:
- A peer cannot be both `rr-client` and `rs-client`
- `rs-client` is incompatible with `add-path-receive` (route server uses send-only ADD-PATH per RFC 7947)

### enforce-first-as

When `true` (default), bgpgg rejects UPDATE messages where the first AS in AS_PATH does not match the peer's configured ASN (RFC 4271 Section 6.3). Set to `false` to disable this check.

### md5-key-file

Path to a file containing the TCP MD5 key (RFC 2385). The file must be readable by the daemon and should be mode `600`. The key is read as a text string with leading/trailing whitespace trimmed.

```yaml
peers:
  - address: "192.168.1.1"
    md5-key-file: "/etc/bgpgg/peer1.key"
```

### next-hop-self

When `true`, bgpgg rewrites the NEXT_HOP attribute to its own local address when advertising routes to this peer. Useful for iBGP peers that do not have a route to the original NEXT_HOP.

## BMP Configuration

Optional. Monitor BGP with external collectors:

```yaml
bmp_servers:
  - address: "127.0.0.1:11019" # Required if using BMP
    statistics_timeout: 60     # Optional: Stats interval in seconds (0 to disable)
```
