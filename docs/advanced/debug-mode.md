The debug mode exposes the following endpoints:

| Path                                                            | Method           | Description                               |
|-----------------------------------------------------------------|------------------|-------------------------------------------|
| `/debug/vars`                                                   |     `GET`        |  Go expvars                               |
| `/debug/pprof`                                                  |     `GET`        |  pprof profiling data                     |

Additionally, the log level will be set to DEBUG.
 
## Enabling the Debug Mode

To enable the debug mode, you need to enable Traefik's API.

??? abstract "Using the Command Line"

    Option | Values | Default Value 
    -- | -- | --:
    --api | \[true\|false\] | false
    --api.debug | \[true\|false\] | false 
    
    {!more-on-command-line.md!}

??? abstract "Using the Configuration File"

    ```toml
    [api] 
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
    
    {!more-on-configuration-file.md!}

??? abstract "Using a Key/Value Store"

    Key | Values | Default Value
    -- | -- | --:
    api | \[true\|false\] | false
    api.debug | \[true\|false\] | false 
    
    {!more-on-key-value-store.md!}
    