# Keepalive

Keepalive option indicates whether `SO_KEEPALIVE` on the socket should be enabled.

## Keepalive configuration

Keepalive configuration can be set for all $productName$ mappings in the [`ambassador Module`](../../running/ambassador) or set per [`AmbassadorMapping`](../mappings#configuring-ambassadormappings).

The `keepalive` attribute configures keepalive. The following fields are supported:

```yaml
keepalive:
  time: <integer>
  interval: <integer>
  probes: <integer>
```

### `time`

(Default: `7200`) The number of seconds a connection needs to be idle before keep-alive probes start being sent.

### `interval`

(Default: `75`) The number of seconds between keep-alive probes.

### `probes`

(Default: `9`) is the maximum number of keepalive probes to send without response before deciding the connection is dead.

## Examples

Keepalive probes defined on a single mapping:

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind:  AmbassadorMapping
metadata:
  name:  quote-backend
spec:
  prefix: /backend/
  service: quote
  keepalive:
    time: 100
    interval: 10
    probes: 9
```

A global keepalive configuration:

```yaml
apiVersion: getambassador.io/v2
kind:  Module
metadata:
  name:  ambassador
spec:
  config:
    keepalive:
       time: 100
       interval: 10
       probes: 9
---
apiVersion: x.getambassador.io/v3alpha1
kind:  AmbassadorMapping
metadata:
  name:  quote-backend
spec:
  prefix: /backend/
  service: quote
```
