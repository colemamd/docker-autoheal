# Docker Autoheal (based on willfarrel/autoheal)

Monitor and restart unhealthy docker containers.
This functionality was proposed to be included with the addition of `HEALTHCHECK`, however didn't make the cut.
This container is a stand-in till there is native support for `--exit-on-unhealthy` <https://github.com/docker/docker/pull/22719>.

## Supported tags and Dockerfile links

- [`latest` (*Dockerfile*)](https://github.com/colemamd/docker-autoheal/blob/main/Dockerfile)
- [`v0.1` (*Dockerfile*)](https://github.com/colemamd/docker-autoheal/blob/main/Dockerfile)

![build status](https://img.shields.io/github/workflow/status/colemamd/docker-autoheal/build)
![docker pulls](https://img.shields.io/docker/pulls/colemamd/autoheal "Total docker pulls")
![image size](https://img.shields.io/docker/image-size/colemamd/autoheal/latest "Total image size")
![mit license](https://img.shields.io/github/license/colemamd/docker-autoheal)

## How to use

### UNIX socket passthrough

```bash
docker run -d \
    --name autoheal \
    --restart=always \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -v /var/run/docker.sock:/var/run/docker.sock \
    colemamd/autoheal
```

### TCP socket

```bash
docker run -d \
    --name autoheal \
    --restart=always \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -e DOCKER_SOCK=tcp://HOST:PORT \
    -v /path/to/certs/:/certs/:ro \
    colemamd/autoheal
```

a) Apply the label `autoheal=true` to your container to have it watched.

b) Set ENV `AUTOHEAL_CONTAINER_LABEL=all` to watch all running containers.

c) Set ENV `AUTOHEAL_CONTAINER_LABEL` to existing label name that has the value `true`.

Note: You must apply `HEALTHCHECK` to your docker images first. See <https://docs.docker.com/engine/reference/builder/#healthcheck> for details.
See <https://docs.docker.com/engine/security/https/> for how to configure TCP with mTLS

The certificates, and keys need these names:

- ca.pem
- client-cert.pem
- client-key.pem

### Change Timezone

If you need the timezone to match the local machine, you can map the `/etc/localtime` into the container.

```bash
docker run ... -v /etc/localtime:/etc/localtime:ro
```

## ENV Defaults

```bash
AUTOHEAL_CONTAINER_LABEL=autoheal
AUTOHEAL_INTERVAL=5   # check every 5 seconds
AUTOHEAL_START_PERIOD=0   # wait 0 seconds before first health check
AUTOHEAL_DEFAULT_STOP_TIMEOUT=10   # Docker waits max 10 seconds (the Docker default) for a container to stop before killing during restarts (container overridable via label, see below)
DOCKER_SOCK=/var/run/docker.sock   # Unix socket for curl requests to Docker API
CURL_TIMEOUT=30     # --max-time seconds for curl requests to Docker API
```

### Optional Container Labels

```bash
autoheal.stop.timeout=20        # Per containers override for stop timeout seconds during restart
```

## Testing

```bash
docker build -t autoheal .

docker run -d \
    -e AUTOHEAL_CONTAINER_LABEL=all \
    -v /var/run/docker.sock:/var/run/docker.sock \
    autoheal                                                                        
```

## Apprise Notifications

(Currently only Discord notifications are supported)

Notifications are handled by a separate [apprise container](https://hub.docker.com/r/caronc/apprise).  
Apprise Discord config [documentation](https://github.com/caronc/apprise/wiki/Notify_discord)

```bash
APPRISE_CONTAINER=apprise             #your Apprise container name OR ip_address, default name is apprise
APPRISE_PORT=8000                     #your Apprise port number, default is 8000
DISCORD_WEBHOOK_ID=abcd1234           #your Discord webhook ID
DISCORD_WEBHOOK_TOKEN=ABCD1234        #your Discord webhook token
```
