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
enhanced-rr-stale-ttl: 360            # Optional: RFC 7313 stale route TTL in seconds (default: 360)
llgr:                                 # Optional: RFC 9494 server-level LLGR (peers inherit)
  enabled: true
  stale-time: 0
bgp-ls:                               # Optional: RFC 9552 BGP Link-State
  max-ls-entries: 0                    # Optional: Max BGP-LS NLRIs in Loc-RIB (default: 0, unlimited)
  instance-id: 0                       # Optional: Instance ID for locally originated NLRIs (default: 0)
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
    graceful-shutdown: false         # Optional: RFC 8326 tag routes with GRACEFUL_SHUTDOWN community (default: false)
    ttl-min: null                    # Optional: RFC 5082 GTSM minimum TTL (default: disabled)
    llgr:                            # Optional: RFC 9494 Long-Lived Graceful Restart
      enabled: true                  # Default: true
      stale-time: 0                  # Long-lived stale time in seconds (24-bit max: 16777215)
      afi-safis: []                  # AFI/SAFIs to enable LLGR for
    send-rpki-community: false       # Optional: Attach RPKI origin validation state extended community on export (default: false)
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

### graceful-shutdown

When `true`, bgpgg tags all outbound routes to this peer with the `GRACEFUL_SHUTDOWN` well-known community (`65535:0`) per RFC 8326. Enable this before taking a session down so that peers can prefer alternate paths during maintenance.

### ttl-min

RFC 5082 GTSM (Generalized TTL Security Mechanism). Sets the minimum acceptable TTL on incoming packets. Use `255` for directly connected peers, `254` for peers one hop away, etc. When unset, GTSM is disabled.

```yaml
peers:
  - address: "192.168.1.1"
    ttl-min: 255
```

### llgr

RFC 9494 Long-Lived Graceful Restart. Extends graceful restart by keeping stale routes for a longer period after a peer goes down. Requires `graceful-restart` to be enabled.

LLGR can be configured at the server level (all peers inherit) or per-peer (overrides server settings). Set `enabled: false` on a peer to explicitly disable even when server-level LLGR is configured.

```yaml
# Server-level (all peers inherit)
llgr:
  enabled: true
  stale-time: 3600
  afi-safis: ["ipv4-unicast"]

peers:
  - address: "192.168.1.1"
    llgr:                          # Per-peer override
      enabled: true
      stale-time: 7200
```

### enhanced-rr-stale-ttl

RFC 7313 Enhanced Route Refresh. Maximum time in seconds to retain stale routes after receiving a BoRR (Beginning of Route Refresh) message. If the peer does not send an EoRR (End of Route Refresh) within this time, stale routes are removed. Set to `null` to disable the timer (stale routes kept indefinitely until EoRR).

### bgp-ls

RFC 9552 BGP Link-State. Allows BGP to carry network topology information (nodes, links, prefixes) from IGP protocols. Enable on peers by adding AFI 16388 / SAFI 71 to their `afi-safis`.

```yaml
bgp-ls:
  max-ls-entries: 10000
  instance-id: 1

peers:
  - address: "192.168.1.1"
    afi-safis:
      - afi: 16388
        safi: 71
```

### send-rpki-community

When `true`, bgpgg attaches the RPKI Origin Validation State extended community (RFC 8097) to routes advertised to this peer. The community reflects the validation result from the local RPKI cache: Valid, Invalid, or NotFound. Requires at least one RPKI cache to be configured.

## RPKI Configuration

Optional. Connect to RPKI-to-Router (RTR) cache servers for BGP origin validation (RFC 6811, RFC 8210):

```yaml
rpki-caches:
  - address: "10.0.0.2:323"           # Required: Cache address (host:port)
    preference: 1                      # Optional: Preference tier, lower = preferred (default: 0)
    transport: "tcp"                   # Optional: "tcp" (default) or "ssh"
    ssh-username: "rpki"               # Required for SSH transport
    ssh-private-key-file: "/etc/bgpgg/rpki.key"  # Required for SSH transport
    ssh-known-hosts-file: "/etc/bgpgg/known_hosts"  # Optional for SSH transport
    retry-interval: 600                # Optional: Override cache retry interval (seconds)
    refresh-interval: 3600             # Optional: Override cache refresh interval (seconds)
    expire-interval: 7200              # Optional: Override cache expire interval (seconds)
```

### preference

RPKI caches are organized into preference tiers. Only caches in the lowest (most preferred) tier are active at startup. If all caches in the active tier go down, bgpgg fails over to the next tier.

### transport

- `tcp` - Plain TCP connection (default, port 323)
- `ssh` - SSH transport (requires `ssh-username` and `ssh-private-key-file`)

## BMP Configuration

Optional. Monitor BGP with external collectors:

```yaml
bmp_servers:
  - address: "127.0.0.1:11019" # Required if using BMP
    statistics_timeout: 60     # Optional: Stats interval in seconds (0 to disable)
```
