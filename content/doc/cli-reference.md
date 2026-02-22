+++
title = 'CLI Reference'
weight = 5
+++

## Peer Management

```bash
bgpgg peer list                         # List all peers
bgpgg peer show 192.168.1.1            # Show peer details
bgpgg peer add 192.168.1.1 65001       # Add peer
bgpgg peer del 192.168.1.1             # Delete peer
```

## Route Management

```bash
bgpgg global rib show                  # Show all routes
bgpgg global rib add 10.0.0.0/24 --nexthop 192.168.1.1
bgpgg global rib add 10.0.0.0/24 --nexthop 192.168.1.1 \
  --origin incomplete --as-path "100 200 300" \
  --local-pref 150 --med 50 --community 65001:100
bgpgg global rib del 10.0.0.0/24
```

## Server Information

```bash
bgpgg global info                      # ASN, router-id, uptime
bgpgg global summary                   # Statistics
```

## Custom gRPC Address

```bash
bgpgg --addr http://192.168.1.1:50051 peer list
```
