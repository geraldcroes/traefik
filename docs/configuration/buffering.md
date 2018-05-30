In some cases request/buffering can be enabled for a specific backend.
By enabling this, Traefik will read the entire request into memory (possibly buffering large requests into disk) and will reject requests that are over a specified limit.
This may help services deal with large data (multipart/form-data for example) more efficiently and should minimise time spent when sending data to a backend server.

For more information please check [oxy/buffer](http://godoc.org/github.com/vulcand/oxy/buffer) documentation.

Example configuration:

```toml
[backends]
  [backends.backend1]
    [backends.backend1.buffering]
      maxRequestBodyBytes = 10485760  
      memRequestBodyBytes = 2097152  
      maxResponseBodyBytes = 10485760
      memResponseBodyBytes = 2097152
      retryExpression = "IsNetworkError() && Attempts() <= 2"
```