With the REST provider enabled, Traefik exposes a REST API that lets you create / update / delete frontends and backends.

## The REST API Endpoint

| Path                         | Method | Description                            |
|------------------------------|--------|----------------------------------------|
| `/api/providers/rest`        | `PUT`  | Updates the provider configuration [^1]|

!!! tip "Did You Know?"
    The Json format is similar to the one used in the [Traefik API](/advanced/api/).    

??? example
    The following will result in configuring a frontend `exposed-api.localhost` that routes the requests to a service hosted on `private-service.localhost`.
   
     ```shell
     curl -XPUT @json_configuration_file "http://localhost:8080/api/providers/rest"
     ```
    
     Content of @json_configuration_file
     ```json
     {
         "frontends": {
           "exposed-api-frontend": {
             "routes": {
               "domain-name": {
                 "rule": "host:exposed-api.localhost"
               }
             },
             "backend": "private-service"
           },
         },
         "backends": {
           "private-service-backend": {
             "servers": {
               "main-server": {
                "URL": "http://private-service.localhost"
               }, 
             }
           }
         }
     }
     ```
## The Json File Format

Below is an example of a more complex backends and frontends structure.

You'll see backends with multiple servers that are load balanced, and a circuit breaker.

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

## Customizing the Entrypoint

By default, Traefik uses an entrypoint named `traefik` to listen for REST API requests. 
You can change this by in the `entryPoint` option (see the details in [Enabling the REST Provider](#enabling-the-rest-provider)).

## Enabling the REST Provider

??? abstract "Using the Command Line"

    Option | Default Value | Description 
    -- | -- | --:
    --rest | | Enables the REST Provider
    --rest.entrypoint | traefik | The entrypoint the API will be available on
    
    {!more-on-command-line.md!}

??? abstract "Using the Configuration File"

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

??? abstract "Using a Key/Value Store"

    Option | Default Value | Description 
    -- | -- | --:
    rest | | Enables the REST Provider
    rest.entrypoint | traefik | The entrypoint the API will be available on
    
    {!more-on-key-value-store.md!}

[^1]: For compatibility reasons, when you activate the REST provider, you can also use the deprecated `/api/providers/web` endpoint.