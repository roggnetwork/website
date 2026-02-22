+++
title = 'Policy'
weight = 4
+++

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
