Traefik consists of a single binary.
There are multiple ways of downloading and running it.

??? abstract "Using The Official Binary File"

    Download the latest binary from the [releases](https://github.com/containous/traefik/releases) page, then run Traefik.
        
    ```shell
        ./traefik -c traefik.toml
    ```
    
    You can use the provided [sample configuration file](https://raw.githubusercontent.com/containous/traefik/master/traefik.sample.toml) and build from it.


??? abstract "Using The Official Docker Image"

    ```shell
    docker run -d -p 8080:8080 -p 80:80 -v $PWD/traefik.toml:/etc/traefik/traefik.toml traefik
    ```
    
    You can use the provided [sample configuration file](https://raw.githubusercontent.com/containous/traefik/master/traefik.sample.toml) and build from it.
    
??? abstract "Using the Source Code"

    Traefik is an open-source project, you can [download the source](https://github.com/containous/traefik/) from the official Github page and build the binary.
        
    For more details on building the binary, refer to the [contributing section](/contributing/contributing) of the documentation.