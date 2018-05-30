In a micro-service architecture, with a central service discovery, setting constraints limits Traefik scope to a smaller number of routes.

Traefik filters services according to service attributes/tags set in your configuration backends.

Supported filters:

- `tag`

## Simple

```toml
# Simple matching constraint
constraints = ["tag==api"]

# Simple mismatching constraint
constraints = ["tag!=api"]

# Globbing
constraints = ["tag==us-*"]
```

## Multiple

```toml
# Multiple constraints
#   - "tag==" must match with at least one tag
#   - "tag!=" must match with none of tags
constraints = ["tag!=us-*", "tag!=asia-*"]
```

### Backend-specific

Supported backends:

- Docker
- Consul K/V
- BoltDB
- Zookeeper
- Etcd
- Consul Catalog
- Rancher
- Marathon
- Kubernetes (using a provider-specific mechanism based on label selectors)

```toml
# Backend-specific constraint
[consulCatalog]
# ...
constraints = ["tag==api"]

# Backend-specific constraint
[marathon]
# ...
constraints = ["tag==api", "tag!=v*-beta"]
``