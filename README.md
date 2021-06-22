# Docker Registry

An on-premises solution for docker registry.

## Registry Proxy Configurations

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

# If REGISTRY\_ADDRESS is a url, it could only be the root address without
  specifying any path e.g. http://172.16.17.38 is accepted but
  http://172.16.17.38/v2 is not an accepted mirror url.

# Port number could be used if the registry is not on the root address e.g.
  http://172.16.17.38:5000 is an accepted mirror address.
