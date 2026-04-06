+++
title = 'CLI Reference'
weight = 5
+++

## Peer Management

```bash
bgpgg peer list                         # List all peers
bgpgg peer show 192.168.1.1            # Show peer details
bgpgg peer add 192.168.1.1 65001       # Add peer
bgpgg peer add 192.168.1.1 65001 \
  --port 1179 \
  --max-prefix-limit 1000 \
  --max-prefix-action discard \
  --md5-key-file /etc/bgpgg/peer.key \ # TCP MD5 signature (RFC 2385)
  --next-hop-self                       # Rewrite NEXT_HOP to local address
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

## RPKI Management

```bash
bgpgg rpki list                        # List configured RPKI caches and VRP counts
bgpgg rpki add 10.0.0.2:323           # Add RPKI cache (TCP)
bgpgg rpki add 10.0.0.2:22 \
  --preference 5 \
  --transport ssh \
  --ssh-username rpki \
  --ssh-private-key-file /etc/bgpgg/rpki.key \
  --ssh-known-hosts-file /etc/bgpgg/known_hosts \
  --retry-interval 600 \
  --refresh-interval 3600 \
  --expire-interval 7200
bgpgg rpki del 10.0.0.2:323           # Remove RPKI cache
bgpgg rpki validate 10.0.0.0/24 65001 # Validate prefix origin
```

## BGP-LS Management

```bash
bgpgg global rib add-ls node \
  --asn 65000 \
  --router-id 10.0.0.1 \
  --protocol ospfv2 \
  --name "router1"                     # Add BGP-LS node NLRI
bgpgg global rib del-ls node \
  --asn 65000 \
  --router-id 10.0.0.1 \
  --protocol ospfv2                    # Delete BGP-LS node NLRI
```

Protocol IDs: `isis-l1`, `isis-l2`, `ospfv2`, `direct`, `static`, `ospfv3`

Optional flags: `--identifier <uint64>` (BGP-LS identifier, default: 0)

## Custom gRPC Address

```bash
bgpgg --addr http://192.168.1.1:50051 peer list
```
