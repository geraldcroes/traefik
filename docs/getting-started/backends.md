A backend is responsible to load-balance the traffic coming from one or more frontends to a set of http servers.

#### Servers

Servers are simply defined using a `url`. You can also apply a custom `weight` to each server (this will be used by load-balancing).

!!! note
    Paths in `url` are ignored. Use `Modifier` to specify paths instead.

Here is an example of backends and servers definition:

```toml
[backends]
  [backends.backend1]
    # ...
    [backends.backend1.servers.server1]
    url = "http://172.17.0.2:80"
    weight = 10
    [backends.backend1.servers.server2]
    url = "http://172.17.0.3:80"
    weight = 1
  [backends.backend2]
    # ...
    [backends.backend2.servers.server1]
    url = "http://172.17.0.4:80"
    weight = 1
    [backends.backend2.servers.server2]
    url = "http://172.17.0.5:80"
    weight = 2
```

- Two backends are defined: `backend1` and `backend2`
- `backend1` will forward the traffic to two servers: `http://172.17.0.2:80"` with weight `10` and `http://172.17.0.3:80` with weight `1`.
- `backend2` will forward the traffic to two servers: `http://172.17.0.4:80"` with weight `1` and `http://172.17.0.5:80` with weight `2`.

#### Load-balancing

Various methods of load-balancing are supported:

- `wrr`: Weighted Round Robin.
- `drr`: Dynamic Round Robin: increases weights on servers that perform better than others.
    It also rolls back to original weights if the servers have changed.

#### Circuit breakers

A circuit breaker can also be applied to a backend, preventing high loads on failing servers.
Initial state is Standby. CB observes the statistics and does not modify the request.
In case the condition matches, CB enters Tripped state, where it responds with predefined code or redirects to another frontend.
Once Tripped timer expires, CB enters Recovering state and resets all stats.
In case the condition does not match and recovery timer expires, CB enters Standby state.

It can be configured using:

- Methods: `LatencyAtQuantileMS`, `NetworkErrorRatio`, `ResponseCodeRatio`
- Operators:  `AND`, `OR`, `EQ`, `NEQ`, `LT`, `LE`, `GT`, `GE`

For example:

- `NetworkErrorRatio() > 0.5`: watch error ratio over 10 second sliding window for a frontend.
- `LatencyAtQuantileMS(50.0) > 50`:  watch latency at quantile in milliseconds.
- `ResponseCodeRatio(500, 600, 0, 600) > 0.5`: ratio of response codes in ranges [500-600) and [0-600).

Here is an example of backends and servers definition:

```toml
[backends]
  [backends.backend1]
    [backends.backend1.circuitbreaker]
    expression = "NetworkErrorRatio() > 0.5"
    [backends.backend1.servers.server1]
    url = "http://172.17.0.2:80"
    weight = 10
    [backends.backend1.servers.server2]
    url = "http://172.17.0.3:80"
    weight = 1
```

- `backend1` will forward the traffic to two servers: `http://172.17.0.2:80"` with weight `10` and `http://172.17.0.3:80` with weight `1` using default `wrr` load-balancing strategy.
- a circuit breaker is added on `backend1` using the expression `NetworkErrorRatio() > 0.5`: watch error ratio over 10 second sliding window

#### Maximum connections

To proactively prevent backends from being overwhelmed with high load, a maximum connection limit can also be applied to each backend.

Maximum connections can be configured by specifying an integer value for `maxconn.amount` and `maxconn.extractorfunc` which is a strategy used to determine how to categorize requests in order to evaluate the maximum connections.

For example:
```toml
[backends]
  [backends.backend1]
    [backends.backend1.maxconn]
       amount = 10
       extractorfunc = "request.host"
   # ...
```

- `backend1` will return `HTTP code 429 Too Many Requests` if there are already 10 requests in progress for the same Host header.
- Another possible value for `extractorfunc` is `client.ip` which will categorize requests based on client source ip.
- Lastly `extractorfunc` can take the value of `request.header.ANY_HEADER` which will categorize requests based on `ANY_HEADER` that you provide.

#### Sticky sessions

Sticky sessions are supported with both load balancers.  
When sticky sessions are enabled, a cookie is set on the initial request.
The default cookie name is an abbreviation of a sha1 (ex: `_1d52e`).
On subsequent requests, the client will be directed to the backend stored in the cookie if it is still healthy.
If not, a new backend will be assigned.

```toml
[backends]
  [backends.backend1]
    # Enable sticky session
    [backends.backend1.loadbalancer.stickiness]

    # Customize the cookie name
    #
    # Optional
    # Default: a sha1 (6 chars)
    #
    #  cookieName = "my_cookie"
```

The deprecated way:

```toml
[backends]
  [backends.backend1]
    [backends.backend1.loadbalancer]
      sticky = true
```

#### Health Check

A health check can be configured in order to remove a backend from LB rotation as long as it keeps returning HTTP status codes other than `2xx` to HTTP GET requests periodically carried out by Traefik.  
The check is defined by a path appended to the backend URL and an interval (given in a format understood by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration)) specifying how often the health check should be executed (the default being 30 seconds).
Each backend must respond to the health check within 5 seconds.  
By default, the port of the backend server is used, however, this may be overridden.

A recovering backend returning `2xx` responses again is being returned to the LB rotation pool.

For example:
```toml
[backends]
  [backends.backend1]
    [backends.backend1.healthcheck]
    path = "/health"
    interval = "10s"
```

To use a different port for the health check:
```toml
[backends]
  [backends.backend1]
    [backends.backend1.healthcheck]
    path = "/health"
    interval = "10s"
    port = 8080
```


To use a different scheme for the health check:
```toml
[backends]
  [backends.backend1]
    [backends.backend1.healthcheck]
    path = "/health"
    interval = "10s"
    scheme = "http"
```

Additional http headers and hostname to health check request can be specified, for instance:
```toml
[backends]
  [backends.backend1]
    [backends.backend1.healthcheck]
    path = "/health"
    interval = "10s"
    hostname = "myhost.com"
    port = 8080
      [backends.backend1.healthcheck.headers]
      My-Custom-Header = "foo"
      My-Header = "bar"
```