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
asn: 65000                           # Required: Autonomous System Number
router_id: "1.1.1.1"                 # Required: Router ID (IPv4 address)
listen_addr: "0.0.0.0:179"           # Optional: BGP listen address (default: 0.0.0.0:179)
grpc_listen_addr: "127.0.0.1:50051"  # Optional: gRPC API address (default: 127.0.0.1:50051)
log_level: "info"                    # Optional: Log level (default: info)
hold_time_secs: 180                  # Optional: BGP hold time (default: 180)
connect_retry_secs: 30               # Optional: Connection retry interval (default: 30)
cluster-id: "1.1.1.1"                # Optional: Route reflector cluster ID (defaults to router_id)
sys_name: "bgpgg router"             # Optional: BMP system name
sys_descr: "BGP daemon"              # Optional: BMP system description
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
- `BGPGG_ROUTER_ID` - Router ID
- `BGPGG_LISTEN_ADDR` - BGP listen address
- `BGPGG_GRPC_LISTEN_ADDR` - gRPC listen address
- `BGPGG_LOG_LEVEL` - Log level
- `BGPGG_HOLD_TIME_SECS` - BGP hold time
- `BGPGG_CONNECT_RETRY_SECS` - Connection retry interval

## Peer Configuration

Optional. Define BGP peers:

```yaml
peers:
  - address: "192.168.1.1"       # Required: Peer IP address
    asn: 65001                   # Optional: Peer ASN (for validation)
    port: 179                    # Optional: BGP port (default: 179)
    passive_mode: false          # Optional: Don't initiate connection (default: false)
    idle_hold_time_secs: 30      # Optional: Idle hold time (default: 30)
    damp_peer_oscillations: true # Optional: Exponential backoff (default: true)
    max_prefix:                  # Optional: Prefix limit
      limit: 1000
      action: "terminate"
    graceful-restart:            # Optional: GR settings
      enabled: true              # Default: true
      restart_time: 120          # GR time in seconds (default: 120, max: 4095)
    rr-client: false             # Optional: Route Reflector client (default: false)
    add-path-send: "disabled"    # Optional: Add-path send mode (default: disabled)
    add-path-receive: false      # Optional: Accept multiple paths (default: false)
    import-policy: []            # Optional: List of import policy names
    export-policy: []            # Optional: List of export policy names
```

### max_prefix

- `terminate` - Drop the BGP session when limit is exceeded
- `discard` - Silently ignore prefixes beyond the limit, keep session up

### add-path-send

- `disabled` - Send only the best path (default)
- `all` - Advertise all available paths to the peer

## BMP Configuration

Optional. Monitor BGP with external collectors:

```yaml
bmp_servers:
  - address: "127.0.0.1:11019" # Required if using BMP
    statistics_timeout: 60     # Optional: Stats interval in seconds (0 to disable)
```
