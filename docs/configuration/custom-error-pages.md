Custom error pages can be returned in lieu of the default, according to frontend-configured ranges of HTTP Status codes.

In the example below, if a 503 status is returned from the frontend "website", the custom error page at http://2.3.4.5/503.html is returned with the actual status code set in the HTTP header.

!!! note
    The `503.html` page itself is not hosted on Traefik, but some other infrastructure.

```toml
[frontends]
  [frontends.website]
  backend = "website"
  [frontends.website.errors]
    [frontends.website.errors.network]
    status = ["500-599"]
    backend = "error"
    query = "/{status}.html"
  [frontends.website.routes.website]
  rule = "Host: website.mydomain.com"

[backends]
  [backends.website]
    [backends.website.servers.website]
    url = "https://1.2.3.4"
  [backends.error]
    [backends.error.servers.error]
    url = "http://2.3.4.5"
```

In the above example, the error page rendered was based on the status code.
Instead, the query parameter can also be set to some generic error page like so: `query = "/500s.html"`

Now the `500s.html` error page is returned for the configured code range.
The configured status code ranges are inclusive; that is, in the above example, the `500s.html` page will be returned for status codes `500` through, and including, `599`.