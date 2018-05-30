## Health Check Configuration

```toml
# Enable custom health check options.
[healthcheck]

# Set the default health check interval.
#
# Optional
# Default: "30s"
#
# interval = "30s"
```

- `interval` set the default health check interval.  
Will only be effective if health check paths are defined.  
Given provider-specific support, the value may be overridden on a per-backend basis.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).  
If no units are provided, the value is parsed assuming seconds.