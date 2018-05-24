You can configure frontends and backends in Traefik using its RESTful API.

## The REST API Endpoint

| Path                         | Method | Description         |
|------------------------------|--------|---------------------|
| `/api/providers/rest`        | `PUT`  | update provider [^1]|

## The REST API Json

```shell
curl -XPUT @file "http://localhost:8080/api/providers/rest"
```

with `@file`:
```json
{
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
```

## Customizing the REST Provider Entrypoint

By default, Traefik uses an entrypoint named `traefik` to listen for REST API requests.
You can change this by using the `entryPoint` option (see the details in [Enabling the REST Provider](#enabling-the-rest-provider)) 

## Enabling the REST Provider

To enable the debug mode, you need to enable Traefik's API.

??? configuration "Using the Command Line"

    Option | Default Value 
    -- | -- :
    --rest | false
    --rest.entrypoint | traefik
    
    {!more-on-command-line.md!}

??? configuration "Using the Configuration File"

    ```toml
    # Enable rest backend.
    [rest]
        # Name of the related entry point
        #
        # Optional
        # Default: "traefik"
        #
        entryPoint = "traefik"
    ```
    
    {!more-on-configuration-file.md!}

??? configuration "Using a Key/Value Store"

    Key | Default Value
    -- | -- :
    rest | false
    rest.entryPoint | traefik  
    
    {!more-on-key-value-store.md!}


[^1]: For compatibility reasons, when you activate the REST provider, you can also use the deprecated `/api/providers/web` endpoint.