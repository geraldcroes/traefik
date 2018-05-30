## Timeouts

### Responding Timeouts

`respondingTimeouts` are timeouts for incoming requests to the Traefik instance.

```toml
[respondingTimeouts]

# readTimeout is the maximum duration for reading the entire request, including the body.
#
# Optional
# Default: "0s"
#
# readTimeout = "5s"

# writeTimeout is the maximum duration before timing out writes of the response.
#
# Optional
# Default: "0s"
#
# writeTimeout = "5s"

# idleTimeout is the maximum duration an idle (keep-alive) connection will remain idle before closing itself.
#
# Optional
# Default: "180s"
#
# idleTimeout = "360s"
```

- `readTimeout` is the maximum duration for reading the entire request, including the body.  
If zero, no timeout exists.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

- `writeTimeout` is the maximum duration before timing out writes of the response.  
It covers the time from the end of the request header read to the end of the response write.
If zero, no timeout exists.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

- `idleTimeout` is the maximum duration an idle (keep-alive) connection will remain idle before closing itself.  
If zero, no timeout exists.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

### Forwarding Timeouts

`forwardingTimeouts` are timeouts for requests forwarded to the backend servers.

```toml
[forwardingTimeouts]

# dialTimeout is the amount of time to wait until a connection to a backend server can be established.
#
# Optional
# Default: "30s"
#
# dialTimeout = "30s"

# responseHeaderTimeout is the amount of time to wait for a server's response headers after fully writing the request (including its body, if any).
#
# Optional
# Default: "0s"
#
# responseHeaderTimeout = "0s"
```

- `dialTimeout` is the amount of time to wait until a connection to a backend server can be established.  
If zero, no timeout exists.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

- `responseHeaderTimeout` is the amount of time to wait for a server's response headers after fully writing the request (including its body, if any).  
If zero, no timeout exists.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.


### Idle Timeout (deprecated)

Use [respondingTimeouts](/configuration/commons/#responding-timeouts) instead of `idleTimeout`.
In the case both settings are configured, the deprecated option will be overwritten.

`idleTimeout` is the maximum amount of time an idle (keep-alive) connection will remain idle before closing itself.
This is set to enforce closing of stale client connections.

Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

```toml
# idleTimeout
#
# DEPRECATED - see [respondingTimeouts] section.
#
# Optional
# Default: "180s"
#
idleTimeout = "360s"
```