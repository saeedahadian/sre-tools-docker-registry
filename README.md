# Docker Registry

An on-premises solution for docker registry.

## Pull-through Cache Configurations

The default behavior of docker registry is to only serve the images that it
locally has but we can use it as a pull-through cache so that the registry gets
the missing images automatically from Docker hub. This configuration is needed
to set up a pull-through cache.
```yaml
proxy:
  remoteurl: https://registry-1.docker.co
  username: [username]
  password: [password]
```
`username` and `password` fields are not mandatory.

#### Connection refused!

When registry could not connect to docker hub's remote address, either reload
your docker engine or change the DNS of the server to something like Quad9.

```bash
# /etc/resolv.conf

nameserver 9.9.9.9
```

### Using multiple hubs

Some images are either not present or outdated on Docker Hub and we need to
pull them from another registry with a different url. For instance, Elastic
Stack images are maintained on `docker.elastic.co`. For this purpose, we
create a new container to specifically pull images from Elastic hub. We should
also use NginX to route all requests for Elastic images to this particular
container.
```yaml
proxy:
  remoteurl: https://docker.elastic.co
```

## Proxy Configurations

In case a proxy access was needed for docker registry to work we can use
this configuration in `~/.docker/config.json`.
```json
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://127.0.0.1:3001",
     "httpsProxy": "http://127.0.0.1:3001",
     "noProxy": "*.test.example.com,.example2.com,127.0.0.0/8"
   }
 }
}
```

## Server Configurations

All the servers that want to use the docker registry instead of docker hub
should have this daemon configuration in `/etc/docker/daemon.json`.

```json
{
	"registry-mirrors" : [
		"REGISTRY_ADDRESS"
	]
}
```

**Important Notes:**

+ If REGISTRY\_ADDRESS is a url, it could only be the root address without
  specifying any path e.g. http://172.16.17.38 is accepted but
  http://172.16.17.38/v2 is not an accepted mirror url.

+ Port number could be used if the registry is not on the root address e.g.
  http://172.16.17.38:5000 is an accepted mirror address.
