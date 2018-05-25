If enabled, Traefik can expose an API that shows information about its current health on the `/health` endpoint.

!!! tip "Did You Know?"
    It is possible to customize the endpoint of the healthcheck. To learn how, refer to the [Traefik's API documentation](/advanced/api.md).

## Sample Health Check Request

```shell
curl -s "http://localhost:8080/health" | jq .
```

```json
{
  // Traefik PID
  "pid": 2458,
  // Traefik server uptime (formated time)
  "uptime": "39m6.885931127s",
  //  Traefik server uptime in seconds
  "uptime_sec": 2346.885931127,
  // current server date
  "time": "2015-10-07 18:32:24.362238909 +0200 CEST",
  // current server date in seconds
  "unixtime": 1444235544,
  // count HTTP response status code in realtime
  "status_code_count": {
    "502": 1
  },
  // count HTTP response status code since Traefik started
  "total_status_code_count": {
    "200": 7,
    "404": 21,
    "502": 13
  },
  // count HTTP response
  "count": 1,
  // count HTTP response
  "total_count": 41,
  // sum of all response time (formated time)
  "total_response_time": "35.456865605s",
  // sum of all response time in seconds
  "total_response_time_sec": 35.456865605,
  // average response time (formated time)
  "average_response_time": "864.8016ms",
  // average response time in seconds
  "average_response_time_sec": 0.8648016000000001,

  // request statistics [requires --statistics to be set]
  // ten most recent requests with 4xx and 5xx status codes
  "recent_errors": [
    {
      // status code
      "status_code": 500,
      // description of status code
      "status": "Internal Server Error",
      // request HTTP method
      "method": "GET",
      // request hostname
      "host": "localhost",
      // request path
      "path": "/path",
      // RFC 3339 formatted date/time
      "time": "2016-10-21T16:59:15.418495872-07:00"
    }
  ]
}
```

## Enabling the Health Check Endpoint

To enable the health check endpoint, you need to enable Traefik's API.

??? abstract "Using the Command Line"

    Option | Values | Default Value 
    -- | -- | --:
    --api | \[true\|false\] | false 
    
    {!more-on-command-line.md!}

??? abstract "Using the Configuration File"

    ```toml
    [api] 
    ```
    
    {!more-on-configuration-file.md!}

??? abstract "Using a Key/Value Store"

    Key | Values | Default Value
    -- | -- | --:
    api | \[true\|false\] | false
    
    {!more-on-key-value-store.md!}
    
!!! warning
    When enabling the API, [the dashboard](/configuration/dashboard/) is enabled by default. 
    If you don't wan't to enable the dashboard, don't forget to disable it using the `api.dashboard=false` option.

!!! tip "Did You Know?"
    The API provides more than the health check. To learn more about it, refer to the [Traefik's API documentation](/advanced/api.md).