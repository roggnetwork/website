+++
title = 'Architecture'
weight = 1
+++

## Core Design

Built for observability and performance. Async Rust with Tokio runtime. Each peer runs in an independent task - failures are isolated, sessions survive connection drops.

Monitoring (gRPC API, BMP) runs separately from routing. You can query state without impacting packet forwarding.

## Architecture

```text
                        MgmtOp (from CLI/gRPC)
                                  |
                                  v
                         +------------------+
                         |   Server Task    |
                         |                  |
                         | - peers HashMap  |
                         | - route selection|
                         +------------------+
                                 |      ^
                          PeerOp |      | ServerOp
                                 v      |
                         +---------------+
                         |  Peer Task    |
                         | (runs forever)|
                         +---------------+
                                 |
                   +-------------+-------------+
                   |             |             |
                   v             v             v
              +---------+   +---------+   +-----------+
              |  Idle   |   | Connect |   | OpenSent  |
              |  state  |   |  state  |   | OpenConfirm|
              +---------+   +---------+   | Established|
                   |             |        +-----------+
                   |             |             |
              Wait for      Attempt TCP    Handle BGP
              ManualStart   connection     messages
```

Server spawns peer tasks via `spawn_peer()`. Peers run forever (survive TCP failures) and implement the BGP FSM. Server and peers communicate via async channels: `PeerOp` (server → peer), `ServerOp` (peer → server).

## Components

- **bgpggd** - BGP speaker daemon
- **bgpgg** - CLI via gRPC
- **BMP** - Streams BGP events to external collectors

## Routing Tables

- **Adj-RIB-In** - Per-peer incoming routes
- **Loc-RIB** - Global best paths
- **Adj-RIB-Out** - Per-peer outgoing routes (enables ADD-PATH)

## Protocol Support

- **RFC 1997** - BGP Communities Attribute
- **RFC 2918** - Route Refresh Capability for BGP-4
- **RFC 4271** - A Border Gateway Protocol 4 (BGP-4)
- **RFC 4360** - BGP Extended Communities Attribute
- **RFC 4456** - BGP Route Reflection
- **RFC 4724** - Graceful Restart Mechanism for BGP
- **RFC 4760** - Multiprotocol Extensions for BGP-4
- **RFC 6793** - BGP Support for Four-Octet Autonomous System (AS) Number Space
- **RFC 7606** - Revised Error Handling for BGP UPDATE Messages
- **RFC 7854** - BGP Monitoring Protocol (BMP)
- **RFC 7911** - Advertisement of Multiple Paths in BGP (ADD-PATH)
- **RFC 8092** - BGP Large Communities Attribute
