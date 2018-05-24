A frontend consists of a set of rules that determine how incoming requests are forwarded from an entrypoint to a backend.

Rules may be classified in one of two groups: Modifiers and matchers.

#### Modifiers

Modifier rules only modify the request. They do not have any impact on routing decisions being made.

Following is the list of existing modifier rules:

- `AddPrefix: /products`: Add path prefix to the existing request path prior to forwarding the request to the backend.
- `ReplacePath: /serverless-path`: Replaces the path and adds the old path to the `X-Replaced-Path` header. Useful for mapping to AWS Lambda or Google Cloud Functions.
- `ReplacePathRegex: ^/api/v2/(.*) /api/$1`: Replaces the path with a regular expression and adds the old path to the `X-Replaced-Path` header. Separate the regular expression and the replacement by a space.

#### Matchers

Matcher rules determine if a particular request should be forwarded to a backend.

Separate multiple rule values by `,` (comma) in order to enable ANY semantics (i.e., forward a request if any rule matches).
Does not work for `Headers` and `HeadersRegexp`.

Separate multiple rule values by `;` (semicolon) in order to enable ALL semantics (i.e., forward a request if all rules match).

Following is the list of existing matcher rules along with examples:

| Matcher                                                    | Description                                                                                                                                                                                                                                                                             |
|------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Headers: Content-Type, application/json`                  | Match HTTP header. It accepts a comma-separated key/value pair where both key and value must be literals.                                                                                                                                                                               |
| `HeadersRegexp: Content-Type, application/(text/json)`     | Match HTTP header. It accepts a comma-separated key/value pair where the key must be a literal and the value may be a literal or a regular expression.                                                                                                                                  |
| `Host: traefik.io, www.traefik.io`                         | Match request host. It accepts a sequence of literal hosts.                                                                                                                                                                                                                             |
| `HostRegexp: traefik.io, {subdomain:[a-z]+}.traefik.io`    | Match request host. It accepts a sequence of literal and regular expression hosts.                                                                                                                                                                                                      |
| `Method: GET, POST, PUT`                                   | Match request HTTP method. It accepts a sequence of HTTP methods.                                                                                                                                                                                                                       |
| `Path: /products/, /articles/{category}/{id:[0-9]+}`       | Match exact request path. It accepts a sequence of literal and regular expression paths.                                                                                                                                                                                                |
| `PathStrip: /products/`                                    | Match exact path and strip off the path prior to forwarding the request to the backend. It accepts a sequence of literal paths.                                                                                                                                                         |
| `PathStripRegex: /articles/{category}/{id:[0-9]+}`         | Match exact path and strip off the path prior to forwarding the request to the backend. It accepts a sequence of literal and regular expression paths.                                                                                                                                  |
| `PathPrefix: /products/, /articles/{category}/{id:[0-9]+}` | Match request prefix path. It accepts a sequence of literal and regular expression prefix paths.                                                                                                                                                                                        |
| `PathPrefixStrip: /products/`                              | Match request prefix path and strip off the path prefix prior to forwarding the request to the backend. It accepts a sequence of literal prefix paths. Starting with Traefik 1.3, the stripped prefix path will be available in the `X-Forwarded-Prefix` header.                        |
| `PathPrefixStripRegex: /articles/{category}/{id:[0-9]+}`   | Match request prefix path and strip off the path prefix prior to forwarding the request to the backend. It accepts a sequence of literal and regular expression prefix paths. Starting with Traefik 1.3, the stripped prefix path will be available in the `X-Forwarded-Prefix` header. |
| `Query: foo=bar, bar=baz`                                  | Match Query String parameters. It accepts a sequence of key=value pairs.                                                                                                                                                                                                                |

In order to use regular expressions with Host and Path matchers, you must declare an arbitrarily named variable followed by the colon-separated regular expression, all enclosed in curly braces. Any pattern supported by [Go's regexp package](https://golang.org/pkg/regexp/) may be used (example: `/posts/{id:[0-9]+}`).

!!! note
    The variable has no special meaning; however, it is required by the [gorilla/mux](https://github.com/gorilla/mux) dependency which embeds the regular expression and defines the syntax.

You can optionally enable `passHostHeader` to forward client `Host` header to the backend.
You can also optionally enable `passTLSCert` to forward TLS Client certificates to the backend.

##### Path Matcher Usage Guidelines

This section explains when to use the various path matchers.

Use `Path` if your backend listens on the exact path only. For instance, `Path: /products` would match `/products` but not `/products/shoes`.

Use a `*Prefix*` matcher if your backend listens on a particular base path but also serves requests on sub-paths.
For instance, `PathPrefix: /products` would match `/products` but also `/products/shoes` and `/products/shirts`.
Since the path is forwarded as-is, your backend is expected to listen on `/products`.

Use a `*Strip` matcher if your backend listens on the root path (`/`) but should be routeable on a specific prefix.
For instance, `PathPrefixStrip: /products` would match `/products` but also `/products/shoes` and `/products/shirts`.  
Since the path is stripped prior to forwarding, your backend is expected to listen on `/`.  
If your backend is serving assets (e.g., images or Javascript files), chances are it must return properly constructed relative URLs.  
Continuing on the example, the backend should return `/products/shoes/image.png` (and not `/images.png` which Traefik would likely not be able to associate with the same backend).  
The `X-Forwarded-Prefix` header (available since Traefik 1.3) can be queried to build such URLs dynamically.

Instead of distinguishing your backends by path only, you can add a Host matcher to the mix.
That way, namespacing of your backends happens on the basis of hosts in addition to paths.

#### Examples

Here is an example of frontends definition:

```toml
[frontends]
  [frontends.frontend1]
  backend = "backend2"
    [frontends.frontend1.routes.test_1]
    rule = "Host:test.localhost,test2.localhost"
  [frontends.frontend2]
  backend = "backend1"
  passHostHeader = true
  passTLSCert = true
  priority = 10
  entrypoints = ["https"] # overrides defaultEntryPoints
    [frontends.frontend2.routes.test_1]
    rule = "HostRegexp:localhost,{subdomain:[a-z]+}.localhost"
  [frontends.frontend3]
  backend = "backend2"
    [frontends.frontend3.routes.test_1]
    rule = "Host:test3.localhost;Path:/test"
```

- Three frontends are defined: `frontend1`, `frontend2` and `frontend3`
- `frontend1` will forward the traffic to the `backend2` if the rule `Host:test.localhost,test2.localhost` is matched
- `frontend2` will forward the traffic to the `backend1` if the rule `HostRegexp:localhost,{subdomain:[a-z]+}.localhost` is matched (forwarding client `Host` header to the backend)
- `frontend3` will forward the traffic to the `backend2` if the rules `Host:test3.localhost` **AND** `Path:/test` are matched

#### Combining multiple rules

As seen in the previous example, you can combine multiple rules.
In TOML file, you can use multiple routes:

```toml
  [frontends.frontend3]
  backend = "backend2"
    [frontends.frontend3.routes.test_1]
    rule = "Host:test3.localhost"
    [frontends.frontend3.routes.test_2]
    rule = "Path:/test"
```

Here `frontend3` will forward the traffic to the `backend2` if the rules `Host:test3.localhost` **AND** `Path:/test` are matched.

You can also use the notation using a `;` separator, same result:

```toml
  [frontends.frontend3]
  backend = "backend2"
    [frontends.frontend3.routes.test_1]
    rule = "Host:test3.localhost;Path:/test"
```

Finally, you can create a rule to bind multiple domains or Path to a frontend, using the `,` separator:

```toml
 [frontends.frontend2]
    [frontends.frontend2.routes.test_1]
    rule = "Host:test1.localhost,test2.localhost"
  [frontends.frontend3]
  backend = "backend2"
    [frontends.frontend3.routes.test_1]
    rule = "Path:/test1,/test2"
```

#### Rules Order

When combining `Modifier` rules with `Matcher` rules, it is important to remember that `Modifier` rules **ALWAYS** apply after the `Matcher` rules.

The following rules are both `Matchers` and `Modifiers`, so the `Matcher` portion of the rule will apply first, and the `Modifier` will apply later.

- `PathStrip`
- `PathStripRegex`
- `PathPrefixStrip`
- `PathPrefixStripRegex`

`Modifiers` will be applied in a pre-determined order regardless of their order in the `rule` configuration section.

1. `PathStrip`
2. `PathPrefixStrip`
3. `PathStripRegex`
4. `PathPrefixStripRegex`
5. `AddPrefix`
6. `ReplacePath`

#### Priorities

By default, routes will be sorted (in descending order) using rules length (to avoid path overlap):
`PathPrefix:/foo;Host:foo.com` (length == 28) will be matched before `PathPrefixStrip:/foobar` (length == 23) will be matched before `PathPrefix:/foo,/bar` (length == 20).

You can customize priority by frontend. The priority value override the rule length during sorting:

```toml
  [frontends]
    [frontends.frontend1]
    backend = "backend1"
    priority = 20
    passHostHeader = true
      [frontends.frontend1.routes.test_1]
      rule = "PathPrefix:/to"
    [frontends.frontend2]
    backend = "backend2"
    passHostHeader = true
      [frontends.frontend2.routes.test_1]
      rule = "PathPrefix:/toto"
```

Here, `frontend1` will be matched before `frontend2` (`20 > 16`).

#### Custom headers

Custom headers can be configured through the frontends, to add headers to either requests or responses that match the frontend's rules.
This allows for setting headers such as `X-Script-Name` to be added to the request, or custom headers to be added to the response.

!!! warning
    If the custom header name is the same as one header name of the request or response, it will be replaced.

In this example, all matches to the path `/cheese` will have the `X-Script-Name` header added to the proxied request and the `X-Custom-Response-Header` header added to the response.

```toml
[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.headers.customresponseheaders]
    X-Custom-Response-Header = "True"
    [frontends.frontend1.headers.customrequestheaders]
    X-Script-Name = "test"
    [frontends.frontend1.routes.test_1]
    rule = "PathPrefixStrip:/cheese"
```

In this second  example, all matches to the path `/cheese` will have the `X-Script-Name` header added to the proxied request, the `X-Custom-Request-Header` header removed from the request, and the `X-Custom-Response-Header` header removed from the response.

```toml
[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.headers.customresponseheaders]
    X-Custom-Response-Header = ""
    [frontends.frontend1.headers.customrequestheaders]
    X-Script-Name = "test"
    X-Custom-Request-Header = ""
    [frontends.frontend1.routes.test_1]
    rule = "PathPrefixStrip:/cheese"
```

#### Security headers

Security related headers (HSTS headers, SSL redirection, Browser XSS filter, etc) can be added and configured per frontend in a similar manner to the custom headers above.
This functionality allows for some easy security features to quickly be set.

An example of some of the security headers:

```toml
[frontends]
  [frontends.frontend1]
  backend = "backend1"
    [frontends.frontend1.headers]
    FrameDeny = true
    [frontends.frontend1.routes.test_1]
    rule = "PathPrefixStrip:/cheddar"
  [frontends.frontend2]
  backend = "backend2"
    [frontends.frontend2.headers]
    SSLRedirect = true
    [frontends.frontend2.routes.test_1]
    rule = "PathPrefixStrip:/stilton"
```

In this example, traffic routed through the first frontend will have the `X-Frame-Options` header set to `DENY`, and the second will only allow HTTPS request through, otherwise will return a 301 HTTPS redirect.

!!! note
    The detailed documentation for those security headers can be found in [unrolled/secure](https://github.com/unrolled/secure#available-options).
