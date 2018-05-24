### Cluster Leadership

```shell
curl -s "http://localhost:8080/cluster/leader" | jq .
```
```shell
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=UTF-8
< Date: xxx
< Content-Length: 15
```
If the given node is not a cluster leader, an HTTP status of `429-Too-Many-Requests` will be returned.
```json
{
  // current leadership status of the queried node
  "leader": true
}
```