If enabled, Traefik can expose information about currently configured providers, backends, frontends, and routes.

The API also exposes the [dashboard](/configuration/dashboard/), [health metrics](/advanced/health/), and cluster leadership information.

## Available API Endpoints

| Path                                                            | Method           | Description                               |
|-----------------------------------------------------------------|------------------|-------------------------------------------|
| `/`                                                             |     `GET`        | [The dashboard](/configuration/dashboard) |
| `/health`                                                       |     `GET`        | [Health Metrics](/advanced/health/)       |
| `/cluster/leader`                                               |     `GET`        | JSON leader true/false response           |
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

## Customizing the Entrypoint

By default, Traefik uses an entrypoint named `traefik` to listen for API requests. 
You can change this by setting the `entryPoint` option (see the details in [Enabling the API](#enabling-the-api)).

??? example "Example - Defining the Port for the `traefik` Endpoint"

    The following configures the entrypoint `traefik` so it listens on port `8083`.
    
    ```toml
    [entryPoints]
     [entryPoints.traefik]
     address = ":8083"
    ```
    
    

??? example "Example - Setting the API Onto a Different Entrypoint"

    The following will configure an entrypoint named `api-entrypoint` that will listen for traffic on port `8082`.
    Then, it configures the API to be available on the newly created entrypoint.
    
    
    ```toml
    [entryPoints]
     [entryPoints.http]
     address = ":80"
    
     [entryPoints.api-entrypoint]
     address = ":8082"
    
     [entryPoints.another-entrypoint]
     address = ":8083"
        
    [api]
    #configures the API entrypoint to api-entrypoint
    entryPoint = "api-entrypoint"
    ```
    
??? example "Example - Adding Basic Auth to the Entrypoint"
    
    You can apply every configuration option to the API entrypoint, including basic auth.
    
    ```toml    
    [entryPoints]
     [entryPoints.api-entrypoint]
       address=":8080"
       [entryPoints.api-entrypoint.auth]
         [entryPoints.api-entrypoint.auth.basic]
           users = [
             "test:$apr1$H6uskkkW$IgXLP6ewTrSuBkTrqE8wj/",
             "test2:$apr1$d9hr9HBB$4HxwgUir3HP4EsggP/QNo0",
           ]
    
    [api]
    entrypoint="api-entrypoint"
    ```

{!more-on-entrypoints.md!}

## Customizing the Path

Since the API is a backend, you can define a frontend to configure the routes to it.  

??? example "Configuring the API to be Available on `:8080/traefik/`"

    ```toml
    [entryPoints]
      [entryPoints.api-entrypoint]
      address = ":8080"
    
    [api]
    entryPoint="api-entrypoint"
    
    [file]
      [frontends]
        [frontends.api-frontend]
        entryPoints = ["api-entrypoint"]
        backend = "api-backend"
        [frontends.api-frontend.routes.traefik-path]
          rule = "PathPrefix:/traefik/"
      [backends]
        [backends.api-backend]
          [backends.api-backend.servers.traefik-instance]
          url = "http://127.0.0.1:8081"
    
    ```
    
    !!! info "The File Provier"
        In the above example, we've used a file provider to define specific frontends and backends for the API.
        Your can learn about this provider in the [File Provider](/configuration/providers/file/) documentation.


{!more-on-frontends.md!}

### The JSon Response Format

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