# The $productName$ container

To give you flexibility and independence from a hosting platform's uptime, you can pull the `ambassador` and `aes` images from any of the following registries:
- `docker.io/datawire/`
   * **Note**: In rare occasions, you may experience rate limits when using Docker Hub. See [this page](https://www.docker.com/increase-rate-limits) to learn how to deal with them.
- `quay.io/datawire/`
- `gcr.io/datawire/`

For an even more robust installation, consider using a [local registry as a pull through cache](https://docs.docker.com/registry/recipes/mirror/) or configure a [publicly accessible mirror](https://cloud.google.com/container-registry/docs/using-dockerhub-mirroring).

## Environment variables

Use the following variables for the environment of your $productName$ container:

| Purpose                           | Variable                                                                                 | Default value                                       | Value type |
|-----------------------------------|----------------------------------------------------------------------------------------- |-----------------------------------------------------|-------------------------------------------------------------------------------|
| Core                              | `AMBASSADOR_ID`                                                                          | `default`                                           | Plain string |
| Core                              | `AMBASSADOR_NAMESPACE`                                                                   | `default` ([^1])                                    | Kubernetes namespace |
| Core                              | `AMBASSADOR_SINGLE_NAMESPACE`                                                            | Empty                                               | Boolean; non-empty=true, empty=false |
| Core                              | `AMBASSADOR_ENVOY_BASE_ID`                                                               | `0`                                                 | Integer |
| Core                              | `AMBASSADOR_LEGACY_MODE`                                                                 | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Core                              | `AMBASSADOR_FAST_RECONFIGURE`                                                            | `false`                                             | EXPERIMENTAL -- Boolean; `true`=true, any other value=false |
| Core                              | `AMBASSADOR_ENVOY_API_VERSION`                                                           | `V2`                                                | String Enum; `V3` or `V2` |
| Core                              | `AMBASSADOR_UPDATE_MAPPING_STATUS`                                                       | `false`                                             | Boolean; `true`=true, any other value=false |
| Core                              | `AMBASSADOR_DISABLE_SNAPSHOT_SERVER`                                                     | `false`                                             | Boolean; non-empty=true, empty=false |
| Core                              | `AMBASSADOR_JSON_LOGGING`                                                                | `false`                                             | Boolean; non-empty=true, empty=false |
| Core                              | `AMBASSADOR_AMBEX_SNAPSHOT_COUNT`                                                        | `30`                                                | Integer; 0 value disables ambex snapshots |
| Core                              | `AMBASSADOR_AMBEX_NO_RATELIMIT`                                                          | `false`                                             | Boolean; set to `true` to turn disable ratelimiting Envoy reconfiguration |
| $AESproductName$                        | [`AES_LOG_LEVEL`](/docs/edge-stack/latest/topics/running/aes-extensions/#aes_log_level)                                      | `warn`                                              | Log level |
| $AESproductName$                        | [`AES_RATELIMIT_PREVIEW`](/docs/edge-stack/latest/topics/running/aes-extensions/ratelimit#aes-ratelimit_preview)         | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| $AESproductName$                        | [`AES_AUTH_TIMEOUT`](/docs/edge-stack/latest/topics/running/aes-extensions/authentication/#timeout-variables)                  | `4s`                                                | Duration; [Go `time.ParseDuration`][]
| Primary Redis (L4)                | [`REDIS_SOCKET_TYPE`](/docs/edge-stack/latest/topics/running/aes-redis#socket_type)                                          | `tcp`                                               | Go network such as `tcp` or `unix`; see [Go `net.Dial`][] |
| Primary Redis (L4)                | [`REDIS_URL`](/docs/edge-stack/latest/topics/running/aes-redis#url)                                                          | None, must be set explicitly                        | Go network address; for TCP this is a `host:port` pair; see [Go `net.Dial`][] |
| Primary Redis (L4)                | [`REDIS_TLS_ENABLED`](/docs/edge-stack/latest/topics/running/aes-redis#tls_enabled)                                          | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Primary Redis (L4)                | [`REDIS_TLS_INSECURE`](/docs/edge-stack/latest/topics/running/aes-redis#tls_insecure)                                        | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Primary Redis (auth)              | [`REDIS_USERNAME`](/docs/edge-stack/latest/topics/running/aes-redis#username)                                                | Empty                                               | Plain string |
| Primary Redis (auth)              | [`REDIS_PASSWORD`](/docs/edge-stack/latest/topics/running/aes-redis#password)                                                | Empty                                               | Plain string |
| Primary Redis (auth)              | [`REDIS_AUTH`](/docs/edge-stack/latest/topics/running/aes-redis#auth)                                                        | Empty                                               | Requires AES_RATELIMIT_PREVIEW; Plain string |
| Primary Redis (tune)              | [`REDIS_POOL_SIZE`](/docs/edge-stack/latest/topics/running/aes-redis#pool_size)                                              | `10`                                                | Integer |
| Primary Redis (tune)              | [`REDIS_PING_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#ping_interval)                                      | `10s`                                               | Duration; [Go `time.ParseDuration`][] |
| Primary Redis (tune)              | [`REDIS_TIMEOUT`](/docs/edge-stack/latest/topics/running/aes-redis#timeout)                                                  | `0s`                                                | Duration; [Go `time.ParseDuration`][] |
| Primary Redis (tune)              | [`REDIS_SURGE_LIMIT_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#surge_limit_interval)                        | `0s`                                                | Duration; [Go `time.ParseDuration`][] |
| Primary Redis (tune)              | [`REDIS_SURGE_LIMIT_AFTER`](/docs/edge-stack/latest/topics/running/aes-redis#surge_limit_after)                              | The value of `REDIS_POOL_SIZE`                      | Integer |
| Primary Redis (tune)              | [`REDIS_SURGE_POOL_SIZE`](/docs/edge-stack/latest/topics/running/aes-redis#surge_pool_size)                                  | `0`                                                 | Integer |
| Primary Redis (tune)              | [`REDIS_SURGE_POOL_DRAIN_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#surge_pool_drain_interval)              | `1m`                                                | Duration; [Go `time.ParseDuration`][] |
| Primary Redis (tune)              | [`REDIS_PIPELINE_WINDOW`](/docs/edge-stack/latest/topics/running/aes-redis#pipeline_window)                                  | `0`                                                 | Requires AES_RATELIMIT_PREVIEW; Duration; [Go `time.ParseDuration`][] |
| Primary Redis (tune)              | [`REDIS_PIPELINE_LIMIT`](/docs/edge-stack/latest/topics/running/aes-redis#pipeline_limit)                                    | `0`                                                 | Requires AES_RATELIMIT_PREVIEW; Integer; [Go `strconv.ParseInt`][] |
| Primary Redis (tune)              | [`REDIS_TYPE`](/docs/edge-stack/latest/topics/running/aes-redis#type)                                                        | `SINGLE`                                            | Requires AES_RATELIMIT_PREVIEW; String; SINGLE, SENTINEL, or CLUSTER |
| Per-Second RateLimit Redis        | [`REDIS_PERSECOND`](/docs/edge-stack/latest/topics/running/aes-redis#redis_persecond)                                        | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Per-Second RateLimit Redis (L4)   | [`REDIS_PERSECOND_SOCKET_TYPE`](/docs/edge-stack/latest/topics/running/aes-redis#socket_type)                                | None, must be set explicitly (if `REDIS_PERSECOND`) | Go network such as `tcp` or `unix`; see [Go `net.Dial`][] |
| Per-Second RateLimit Redis (L4)   | [`REDIS_PERSECOND_URL`](/docs/edge-stack/latest/topics/running/aes-redis#url)                                                | None, must be set explicitly (if `REDIS_PERSECOND`) | Go network address; for TCP this is a `host:port` pair; see [Go `net.Dial`][] |
| Per-Second RateLimit Redis (L4)   | [`REDIS_PERSECOND_TLS_ENABLED`](/docs/edge-stack/latest/topics/running/aes-redis#tls_enabled)                                | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Per-Second RateLimit Redis (L4)   | [`REDIS_PERSECOND_TLS_INSECURE`](/docs/edge-stack/latest/topics/running/aes-redis#tls_insecure)                              | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| Per-Second RateLimit Redis (auth) | [`REDIS_PERSECOND_USERNAME`](/docs/edge-stack/latest/topics/running/aes-redis#username)                                      | Empty                                               | Plain string |
| Per-Second RateLimit Redis (auth) | [`REDIS_PERSECOND_PASSWORD`](/docs/edge-stack/latest/topics/running/aes-redis#password)                                      | Empty                                               | Plain string |
| Per-Second RateLimit Redis (auth) | [`REDIS_PERSECOND_AUTH`](/docs/edge-stack/latest/topics/running/aes-redis#auth)                                              | Empty                                               | Requires AES_RATELIMIT_PREVIEW; Plain string |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_POOL_SIZE`](/docs/edge-stack/latest/topics/running/aes-redis#pool_size)                                    | `10`                                                | Integer |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_PING_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#ping_interval)                            | `10s`                                               | Duration; [Go `time.ParseDuration`][] |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_TIMEOUT`](/docs/edge-stack/latest/topics/running/aes-redis#timeout)                                        | `0s`                                                | Duration; [Go `time.ParseDuration`][] |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_SURGE_LIMIT_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#surge_limit_interval)              | `0s`                                                | Duration; [Go `time.ParseDuration`][] |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_SURGE_LIMIT_AFTER`](/docs/edge-stack/latest/topics/running/aes-redis#surge_limit_after)                    | The value of `REDIS_PERSECOND_POOL_SIZE`            | Integer |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_SURGE_POOL_SIZE`](/docs/edge-stack/latest/topics/running/aes-redis#surge_pool_size)                        | `0`                                                 | Integer |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_SURGE_POOL_DRAIN_INTERVAL`](/docs/edge-stack/latest/topics/running/aes-redis#surge_pool_drain_interval)    | `1m`                                                | Duration; [Go `time.ParseDuration`][] |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_TYPE`](/docs/edge-stack/latest/topics/running/aes-redis#type)                                              | `SINGLE`                                            | Requires AES_RATELIMIT_PREVIEW; String; SINGLE, SENTINEL, or CLUSTER |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_PIPELINE_WINDOW`](/docs/edge-stack/latest/topics/running/aes-redis#pipeline_window)                        | `0`                                                 | Requires AES_RATELIMIT_PREVIEW; Duration; [Go `time.ParseDuration`][] |
| Per-Second RateLimit Redis (tune) | [`REDIS_PERSECOND_PIPELINE_LIMIT`](/docs/edge-stack/latest/topics/running/aes-redis#pipeline_limit)                          | `0`                                                 | Requires AES_RATELIMIT_PREVIEW; Integer |
| RateLimit                         | `EXPIRATION_JITTER_MAX_SECONDS`                                                          | `300`                                               | Integer |
| RateLimit                         | `USE_STATSD`                                                                             | `false`                                             | Boolean; [Go `strconv.ParseBool`][] |
| RateLimit                         | `STATSD_HOST`                                                                            | `localhost`                                         | Hostname |
| RateLimit                         | `STATSD_PORT`                                                                            | `8125`                                              | Integer |
| RateLimit                         | `GOSTATS_FLUSH_INTERVAL_SECONDS`                                                         | `5`                                                 | Integer |
| RateLimit                         | [`LOCAL_CACHE_SIZE_IN_BYTES`](/docs/edge-stack/latest/topics/running/aes-extensions/ratelimit#local_cache_size_in_bytes) | `0`                                                 | Requires AES_RATELIMIT_PREVIEW; Integer |
| RateLimit                         | [`NEAR_LIMIT_RATIO`](/docs/edge-stack/latest/topics/running/aes-extensions/ratelimit#near_limit_ratio)                   | `0.8`                                               | Requires AES_RATELIMIT_PREVIEW; Float; [Go `strconv.ParseFloat`][] || Developer Portal | `AMBASSADOR_URL` | `https://api.example.com` | URL |
| Developer Portal                  | `DEVPORTAL_CONTENT_URL`                                                                  | `https://github.com/datawire/devportal-content`     | git-remote URL |
| Developer Portal                  | `DEVPORTAL_CONTENT_DIR`                                                                  | `/`                                                 | Rooted Git directory |
| Developer Portal                  | `DEVPORTAL_CONTENT_BRANCH`                                                               | `master`                                            | Git branch name |
| Developer Portal                  | `POLL_EVERY_SECS`                                                                        | `60`                                                | Integer |
| Envoy                             | `STATSD_ENABLED`                                                                         | `false`                                             | Boolean; Python `value.lower() == "true"` |
| Envoy                             | `DOGSTATSD`                                                                              | `false`                                             | Boolean; Python `value.lower() == "true"` |
| Envoy                             | `DD_ENTITY_ID`                                                                           | Empty                                               | String |
| Envoy                             | `ENVOY_CONCURRENCY`                                                                      | Empty                                               | Integer

Log level names are case-insensitive.  From least verbose to most
verbose, valid log levels are `error`, `warn`/`warning`, `info`,
`debug`, and `trace`.

## Port assignments

$productName$ uses the following ports to listen for HTTP/HTTPS traffic automatically via TCP:

| Port | Process  | Function                                                |
|------|----------|---------------------------------------------------------|
| 8001 | envoy    | Internal stats, logging, etc.; not exposed outside pod  |
| 8002 | watt     | Internal watt snapshot access; not exposed outside pod  |
| 8003 | ambex    | Internal ambex snapshot access; not exposed outside pod |
| 8004 | diagd    | Internal `diagd` access when `AMBASSADOR_FAST_RECONFIGURE` is set; not exposed outside pod |
| 8005 | snapshot | Exposes a scrubbed $productName$ snapshot outside of the pod |
| 8080 | envoy    | Default HTTP service port                               |
| 8443 | envoy    | Default HTTPS service port                              |
| 8877 | diagd    | Direct access to diagnostics UI; provided by `busyambassador entrypoint` when `AMBASSADOR_FAST_RECONFIGURE` is set |

[^1]: This may change in a future release to reflect the Pods's
      namespace if deployed to a namespace other than `default`.
      https://github.com/emissary-ingress/emissary/issues/1583

[Go `net.Dial`]: https://golang.org/pkg/net/#Dial
[Go `strconv.ParseBool`]: https://golang.org/pkg/strconv/#ParseBool
[Go `time.ParseDuration`]: https://golang.org/pkg/strconv/#ParseDuration
[Redis 6 ACL]: https://redis.io/topics/acl
