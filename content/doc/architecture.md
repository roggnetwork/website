+++
title = 'Architecture'
weight = 3
+++

## Design Principles

bgpgg is built for observability and performance. The core design separates routing logic from monitoring, allowing external tools to query BGP state without impacting router performance.

## Components

### Daemon (bgpggd)

Main BGP speaker process. Manages peer connections and routing table.

### CLI (bgpgg)

Management interface via gRPC API. Runs independently, does not block daemon operations.

### gRPC API

Control plane interface on port 50051 (default). Used by CLI and external management tools.

## Peer Lifecycle

Each peer runs in an independent task, following RFC 4271 FSM states:

```
Idle -> Connect -> OpenSent -> OpenConfirm -> Established
```

Peers remain active regardless of TCP connection state. Failed connections retry with exponential backoff (when `damp_peer_oscillations` enabled).

## Session Types

Determined after OPEN message exchange:

- **eBGP** - Different ASN. Adds AS_PATH prepending, rewrites NEXT_HOP.
- **iBGP** - Same ASN. Preserves NEXT_HOP, processes LOCAL_PREF.

## Routing Tables

### Adj-RIB-In

Per-peer incoming routes. Stores all received routes before policy application.

### Loc-RIB

Global routing table. Best path per prefix after policy evaluation.

### Adj-RIB-Out

Per-peer outgoing routes. Enables ADD-PATH support (RFC 7911) by tracking which paths were sent to each peer.

## ADD-PATH Support

Allows advertising multiple paths for the same prefix:

```yaml
peers:
  - address: "10.0.0.2"
    add-path-send: "all"       # Send all paths (default: disabled)
    add-path-receive: true     # Accept multiple paths
```

Each route carries `local_path_id` and `remote_path_id` for path tracking.

## Connection Collision Detection

When both sides simultaneously connect, server maintains two connection slots per peer:

- `outgoing` - Connection we initiated
- `incoming` - Connection peer initiated

Collision resolved by comparing BGP IDs (RFC 4271 6.8). Higher BGP ID wins.

## Treat-as-Withdraw

RFC 7606 error handling. Instead of session teardown on malformed attributes:

- Critical attributes (ORIGIN, AS_PATH, NEXT_HOP) - Convert UPDATE to withdrawal
- Optional attributes - Discard attribute, keep route

Improves session stability.

## BMP Monitoring

bgpgg acts as BMP client (RFC 7854), exporting BGP state to external collectors:

```yaml
bmp_servers:
  - address: "127.0.0.1:11019"
    statistics_timeout: 60
```

Message batching: accumulates up to 100 messages or 100ms timeout, whichever comes first.

### BMP Message Types

- **Route Monitoring** - Forwards UPDATE messages
- **Peer Up/Down** - Session state changes
- **Statistics Report** - Periodic stats (configurable interval)
- **Initiation/Termination** - Connection lifecycle

When new BMP server connects to running system:

1. Send Initiation
2. Send PeerUp for all established peers
3. Send all routes in Adj-RIB-In
4. Continue with real-time updates

## Route Reflector

Supports RFC 4456 route reflection:

```yaml
cluster-id: "1.1.1.1"

peers:
  - address: "10.0.0.2"
    rr-client: true
```

Clients receive routes from other clients. Non-clients follow standard iBGP rules.

## Graceful Restart

RFC 4724 support enabled by default:

```yaml
peers:
  - address: "10.0.0.2"
    graceful-restart:
      enabled: true
      restart_time: 120      # Max 4095 seconds
```

Routes marked stale during restart, preserved until new paths received or timer expires.

## Message Flow

```
Server Task              Peer Task
     |                       |
     |-- ManualStart ------->|
     |                       |-- TCP connect -->
     |                       |-- send OPEN -->
     |<-- PeerStateChanged --|
     |<-- OpenReceived ------|   (collision check)
     |<-- HandshakeComplete -|
     |                       |-- Established
     |-- SendUpdate -------->|-- send UPDATE -->
     |<-- PeerUpdate --------|<-- receive UPDATE
```

## Performance Design

- **Async I/O** - Tokio runtime, non-blocking operations
- **Per-peer isolation** - Independent tasks, failures contained
- **Batched writes** - BMP messages buffered to minimize syscalls
- **Separate monitoring** - gRPC API and BMP run independently of routing

## Protocol Support

Currently implements:

- RFC 4271 (BGP-4)
- RFC 4724 (Graceful Restart)
- RFC 4456 (Route Reflection)
- RFC 7606 (Treat-as-Withdraw)
- RFC 7854 (BMP)
- RFC 7911 (ADD-PATH)
