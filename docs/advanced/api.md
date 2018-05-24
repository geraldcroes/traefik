If enabled, Traefik can expose information about currently configured providers, backends, frontends, and routes.

## Available API Endpoints

| Path                                                            | Method           | Description                               |
|-----------------------------------------------------------------|------------------|-------------------------------------------|
| `/`                                                             |     `GET`        | [The dashboard](/configuration/dashboard) |
| `/cluster/leader`                                               |     `GET`        | JSON leader true/false response           |
| `/health`                                                       |     `GET`        | [Health Metrics](/advanced/health/)       |
| `/api`                                                          |     `GET`        | Configuration for all providers           |
| `/api/providers`                                                |     `GET`        | Providers list [^1]                       |
| `/api/providers/{provider}`                                     |     `GET`, `PUT` | Get information on the given provider [^2]|
| `/api/providers/{provider}/backends`                            |     `GET`        | List backends                             |
| `/api/providers/{provider}/backends/{backend}`                  |     `GET`        | Get backend                               |
| `/api/providers/{provider}/backends/{backend}/servers`          |     `GET`        | List servers in backend                   |
| `/api/providers/{provider}/backends/{backend}/servers/{server}` |     `GET`        | Get a server in a backend                 |
| `/api/providers/{provider}/frontends`                           |     `GET`        | List frontends                            |
| `/api/providers/{provider}/frontends/{frontend}`                |     `GET`        | Get a frontend                            |
| `/api/providers/{provider}/frontends/{frontend}/routes`         |     `GET`        | List routes in a frontend                 |
| `/api/providers/{provider}/frontends/{frontend}/routes/{route}` |     `GET`        | Get a route in a frontend                 |


```toml
[api]
  # Name of the related entry point
  #
  # Optional
  # Default: "traefik"
  #
  entryPoint = "traefik"

  # Enabled Dashboard
  #
  # Optional
  # Default: true
  #
  dashboard = true

  # Enable debug mode.
  # This will install HTTP handlers to expose Go expvars under /debug/vars and
  # pprof profiling data under /debug/pprof.
  # Additionally, the log level will be set to DEBUG.
  #
  # Optional
  # Default: false
  #
  debug = true
```

For more customization, see [entry points](/configuration/entrypoints/) documentation and [examples](/user-guide/examples/#ping-health-check).

### Address / Port

You can define a custom address/port like this:

```toml
defaultEntryPoints = ["http"]

[entryPoints]
  [entryPoints.http]
  address = ":80"

  [entryPoints.foo]
  address = ":8082"

  [entryPoints.bar]
  address = ":8083"

[ping]
entryPoint = "foo"

[api]
entryPoint = "bar"
```

In the above example, you would access a regular path, administration panel, and health-check as follows:

* Regular path: `http://hostname:80/path`
* Admin Panel: `http://hostname:8083/`
* Ping URL: `http://hostname:8082/ping`

In the above example, it is _very_ important to create a named dedicated entry point, and do **not** include it in `defaultEntryPoints`.
Otherwise, you are likely to expose _all_ services via that entry point.

### Custom Path

You can define a custom path like this:

```toml
defaultEntryPoints = ["http"]

[entryPoints]
  [entryPoints.http]
  address = ":80"

  [entryPoints.foo]
  address = ":8080"

  [entryPoints.bar]
  address = ":8081"

# Activate API and Dashboard
[api]
entryPoint = "bar"
dashboard = true

[file]
  [backends]
    [backends.backend1]
      [backends.backend1.servers.server1]
      url = "http://127.0.0.1:8081"

  [frontends]
    [frontends.frontend1]
    entryPoints = ["foo"]
    backend = "backend1"
      [frontends.frontend1.routes.test_1]
      rule = "PathPrefixStrip:/yourprefix;PathPrefix:/yourprefix"
```

### Authentication

You can define the authentication like this:

```toml
defaultEntryPoints = ["http"]

[entryPoints]
  [entryPoints.http]
  address = ":80"

 [entryPoints.foo]
   address=":8080"
   [entryPoints.foo.auth]
     [entryPoints.foo.auth.basic]
       users = [
         "test:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/",
         "test2:$apr1$d9hr9HBB$4HxwgUir3HP4EsggP/QNo0",
       ]

[api]
entrypoint="foo"
```

For more information, see [entry points](/configuration/entrypoints/) .

### Provider call example

```shell
curl -s "http://localhost:8080/api" | jq .
```
```json
{
  "file": {
    "frontends": {
      "frontend2": {
        "routes": {
          "test_2": {
            "rule": "Path:/test"
          }
        },
        "backend": "backend1"
      },
      "frontend1": {
        "routes": {
          "test_1": {
            "rule": "Host:test.localhost"
          }
        },
        "backend": "backend2"
      }
    },
    "backends": {
      "backend2": {
        "loadBalancer": {
          "method": "drr"
        },
        "servers": {
          "server2": {
            "weight": 2,
            "URL": "http://172.17.0.5:80"
          },
          "server1": {
            "weight": 1,
            "url": "http://172.17.0.4:80"
          }
        }
      },
      "backend1": {
        "loadBalancer": {
          "method": "wrr"
        },
        "circuitBreaker": {
          "expression": "NetworkErrorRatio() > 0.5"
        },
        "servers": {
          "server2": {
            "weight": 1,
            "url": "http://172.17.0.3:80"
          },
          "server1": {
            "weight": 10,
            "url": "http://172.17.0.2:80"
          }
        }
      }
    }
  }
}
```

[^1]: For compatibility reasons, the [REST provider](/configuration/providers/rest/) endpoints will be available both on `/api/providers/web/*` and `/api/providers/rest/*` 

[^2]: For compatibility reasons, the [REST provider](/configuration/providers/rest/) will appear as "web" in the list returned by `/api/providers/`