## Collected Data

**This feature is disabled by default.**

You can read the public proposal on this topic [here](https://github.com/containous/traefik/issues/2369).

### Why ?

In order to help us learn more about how Traefik is being used and improve it, we collect anonymous usage statistics from running instances.
Those data help us prioritize our developments and focus on what's more important (for example, which configuration backend is used and which is not used).

### What ?

Once a day (the first call begins 10 minutes after the start of Traefik), we collect:

- the Traefik version
- a hash of the configuration
- an **anonymous version** of the static configuration:
    - token, user name, password, URL, IP, domain, email, etc, are removed

!!! note
    We do not collect the dynamic configuration (frontends & backends).

!!! note
    We do not collect data behind the scenes to run advertising programs or to sell such data to third-party.

#### Here is an example

- Source configuration:

```toml
[entryPoints]
    [entryPoints.http]
       address = ":80"

[api]

[Docker]
  endpoint = "tcp://10.10.10.10:2375"
  domain = "foo.bir"
  exposedByDefault = true
  swarmMode = true

  [Docker.TLS]
    ca = "dockerCA"
    cert = "dockerCert"
    key = "dockerKey"
    insecureSkipVerify = true

[ECS]
  domain = "foo.bar"
  exposedByDefault = true
  clusters = ["foo-bar"]
  region = "us-west-2"
  accessKeyID = "AccessKeyID"
  secretAccessKey = "SecretAccessKey"
```

- Obfuscated and anonymous configuration:

```toml
[entryPoints]
    [entryPoints.http]
       address = ":80"

[api]

[Docker]
  endpoint = "xxxx"
  domain = "xxxx"
  exposedByDefault = true
  swarmMode = true

  [Docker.TLS]
    ca = "xxxx"
    cert = "xxxx"
    key = "xxxx"
    insecureSkipVerify = false

[ECS]
  domain = "xxxx"
  exposedByDefault = true
  clusters = []
  region = "us-west-2"
  accessKeyID = "xxxx"
  secretAccessKey = "xxxx"
```

### Show me the code !

If you want to dig into more details, here is the source code of the collecting system: [collector.go](https://github.com/containous/traefik/blob/master/collector/collector.go)

By default we anonymize all configuration fields, except fields tagged with `export=true`.

You can check all fields in the [godoc](https://godoc.org/github.com/containous/traefik/configuration#GlobalConfiguration).

### How to enable this ?

You can enable the collecting system by:

- adding this line in the configuration TOML file:

```toml
# Send anonymous usage data
#
# Optional
# Default: false
#
sendAnonymousUsage = true
```

- adding this flag in the CLI:

```bash
./traefik --sendAnonymousUsage=true
```