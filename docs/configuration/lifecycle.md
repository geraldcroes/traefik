## Life Cycle

Controls the behavior of Traefik during the shutdown phase.

```toml
[lifeCycle]

# Duration to keep accepting requests prior to initiating the graceful
# termination period (as defined by the `graceTimeOut` option). This
# option is meant to give downstream load-balancers sufficient time to
# take Traefik out of rotation.
# Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
# If no units are provided, the value is parsed assuming seconds.
# The zero duration disables the request accepting grace period, i.e.,
# Traefik will immediately proceed to the grace period.
#
# Optional
# Default: 0
#
# requestAcceptGraceTimeout = "10s"

# Duration to give active requests a chance to finish before Traefik stops.
# Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
# If no units are provided, the value is parsed assuming seconds.
# Note: in this time frame no new requests are accepted.
#
# Optional
# Default: "10s"
#
# graceTimeOut = "10s"
```
