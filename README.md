# Docker Registry

An on-premises solution for docker registry.

## Using Registry as a Pull-through Cache

The default behavior of docker registry is to only serve the images that it
locally has but we can use it as a pull-through cache so that the registry
downloads missing images automatically from Docker Hub, stores them in their
local disk and then delivers the image back to the client that requested it.
To transfer the registry into a pull-through cache, add this configuration to
the docker registry's configuration file which is located in
`/etc/docker/registry/config.yml`.

```yaml
proxy:
  remoteurl: https://registry-1.docker.co
  username: [username]
  password: [password]
```

`username` and `password` fields are optional.

### Using Multiple Hubs

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

### Pushing Images to Registry

When a registry instance gets the role of _cache_, you no longer can push
images to it. In this case, you can bring up a new registry instance which
shares the _same volume_ as your cache registry so that you can use this new
instance for pushing your own images to docker registry and all your images
will be also accessible through the cache registry because they share the same
volume.

## Client Configuration

In order to get images from docker registry instead of the Docker Hub, you need
to set this configuration inside the `/etc/docker/daemon.json` file. Note that
the `daemon.json` file might not exist already on your system, in that case,
create a new file with these configurations.

```json
{
	"registry-mirrors" : [
		"REGISTRY_ADDRESS_1",
    "REGISTRY_ADDRESS_2"
	]
}
```

**Important Notes:**

+ If REGISTRY\_ADDRESS is a url, it could only be the root address without
  specifying any path e.g. http://172.16.17.38 is accepted but
  http://172.16.17.38/v2 is not an accepted mirror url.

+ Port number could be used if the registry is not on the root address e.g.
  http://172.16.17.38:5000 is an accepted mirror address.

## Common Issues

### System-wide Proxy for Docker

In case a proxy access was needed for docker registry to work we can use this
configuration in `~/.docker/config.json`. It enables proxy for all docker
instances.

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

### Connection refused!

When registry could not connect to docker hub's remote address, either reload
your docker engine or change the DNS of the server to something like Quad9.

```bash
# /etc/resolv.conf

nameserver 9.9.9.9
```
