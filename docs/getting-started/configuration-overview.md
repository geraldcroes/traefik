Draw a diagram with "static configuration" configures dynamic configuration "providers" configures "routes and stuff"

Configuring Traefik  means setting up the dialogue with providers, and defining the entrypoints. 

Every other elements, such as routes, frontends, backends, SSL certificates, ... will be defined dynamically.

You can combine combine multiple sources of configuration, with the following order of precedence:

- [Key-value stores](/basics/#key-value-stores)
- [Command line arguments](/basics/#arguments)
- [Configuration files](/basics/#configuration-file)

It means that command line arguments override configuration file, and key-value store overrides arguments' values.

!!! warning
    the provider-enabling argument parameters (e.g., `--docker`) set all default values for the specific provider.  
    It must not be used if a configuration source with less precedence wants to set a non-default provider value.

## Configuration file

By default, Traefik will look for `traefik.toml` in the following locations:

- `/etc/traefik/`
- `$HOME/.traefik/`
- `.` (_the working directory_)

You can override this by setting a `configFile` argument:

```bash
traefik --configFile=foo/bar/myconfigfile.toml
```

Please refer to the [global configuration](/configuration/commons) section to get documentation on it.

#### Arguments

Each argument (and command) is described in the help section:

```bash
traefik --help
```

Note that all default values will be displayed as well.

#### Key-value stores

Traefik supports several Key-value stores:

- [Consul](https://consul.io)
- [etcd](https://coreos.com/etcd/)
- [ZooKeeper](https://zookeeper.apache.org/)
- [boltdb](https://github.com/boltdb/bolt)

Please refer to the [User Guide Key-value store configuration](/user-guide/kv-config/) section to get documentation on it.

### Dynamic Traefik configuration

The dynamic configuration concerns :

- [Frontends](/basics/#frontends)
- [Backends](/basics/#backends)
- [Servers](/basics/#servers)
- HTTPS Certificates

Traefik can hot-reload those rules which could be provided by [multiple providers](/configuration/commons).

We only need to enable `watch` option to make Traefik watch configuration backend changes and generate its configuration automatically.
Routes to services will be created and updated instantly at any changes.

Please refer to the [configuration backends](/configuration/commons) section to get documentation on it.

## Commands

### traefik

Usage:
```bash
traefik [command] [--flag=flag_argument]
```

List of Traefik available commands with description :

- `version` : Print version
- `storeconfig` : Store the static Traefik configuration into a Key-value stores. Please refer to the [Store Traefik configuration](/user-guide/kv-config/#store-configuration-in-key-value-store) section to get documentation on it.
- `bug`: The easiest way to submit a pre-filled issue.
- `healthcheck`: Calls Traefik `/ping` to check health.

Each command may have related flags.

All those related flags will be displayed with :

```bash
traefik [command] --help
```

Each command is described at the beginning of the help section:

```bash
traefik --help

# or

docker run traefik[:version] --help
# ex: docker run traefik:1.5 --help
```

### Command: bug

Here is the easiest way to submit a pre-filled issue on [Traefik GitHub](https://github.com/containous/traefik).

```bash
traefik bug
```

Watch [this demo](https://www.youtube.com/watch?v=Lyz62L8m93I).

## Global Configuration

### Main Section

```toml

# Periodically check if a new version has been released.
#
# Optional
# Default: true
#
# checkNewVersion = false

# Backends throttle duration.
#
# Optional
# Default: "2s"
#
# providersThrottleDuration = "2s"

# Controls the maximum idle (keep-alive) connections to keep per-host.
#
# Optional
# Default: 200
#
# maxIdleConnsPerHost = 200

# If set to true invalid SSL certificates are accepted for backends.
# This disables detection of man-in-the-middle attacks so should only be used on secure backend networks.
#
# Optional
# Default: false
#
# insecureSkipVerify = true

# Register Certificates in the rootCA.
#
# Optional
# Default: []
#
# rootCAs = [ "/mycert.cert" ]

# Entrypoints to be used by frontends that do not specify any entrypoint.
# Each frontend can specify its own entrypoints.
#
# Optional
# Default: ["http"]
#
# defaultEntryPoints = ["http", "https"]

# Allow the use of 0 as server weight.
# - false: a weight 0 means internally a weight of 1.
# - true: a weight 0 means internally a weight of 0 (a server with a weight of 0 is removed from the available servers).
#
# Optional
# Default: false
#
# AllowMinWeightZero = true
```

- `graceTimeOut`: Duration to give active requests a chance to finish before Traefik stops.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.  
**Note:** in this time frame no new requests are accepted.

- `providersThrottleDuration`: Backends throttle duration: minimum duration in seconds between 2 events from providers before applying a new configuration.
It avoids unnecessary reloads if multiples events are sent in a short amount of time.  
Can be provided in a format supported by [time.ParseDuration](https://golang.org/pkg/time/#ParseDuration) or as raw values (digits).
If no units are provided, the value is parsed assuming seconds.

- `maxIdleConnsPerHost`: Controls the maximum idle (keep-alive) connections to keep per-host.  
If zero, `DefaultMaxIdleConnsPerHost` from the Go standard library net/http module is used.
If you encounter 'too many open files' errors, you can either increase this value or change the `ulimit`.

- `insecureSkipVerify` : If set to true invalid SSL certificates are accepted for backends.  
**Note:** This disables detection of man-in-the-middle attacks so should only be used on secure backend networks.

- `rootCAs`: Register Certificates in the RootCA. This certificates will be use for backends calls.  
**Note** You can use file path or cert content directly

- `defaultEntryPoints`: Entrypoints to be used by frontends that do not specify any entrypoint.  
Each frontend can specify its own entrypoints.