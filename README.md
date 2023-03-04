# Derper

[![docker workflow](https://github.com/superng6/derper-docker/actions/workflows/docker-image.yml/badge.svg)](https://hub.docker.com/r/superng6/derper)
[![docker pulls](https://img.shields.io/docker/pulls/superng6/derper.svg?color=brightgreen)](https://hub.docker.com/r/superng6/derper)
[![platfrom](https://img.shields.io/badge/platform-amd64%20%7C%20arm64-brightgreen)](https://hub.docker.com/r/superng6/derper/tags)

# Docker Hub
https://hub.docker.com/r/superng6/derper

# Setup

> required: set env `DERP_DOMAIN` to your domain

```bash
docker run -e DERP_DOMAIN=derper.your-domain.com -p 443:443 -p 3478:3478/udp superng6/derper
```

| env                 | required | description                                                            | default value     |
| ------------------- | -------- | ---------------------------------------------------------------------- | ----------------- |
| DERP_DOMAIN         | true     | derper server hostname                                                 | your-hostname.com |
| DERP_CERT_DIR       | false    | directory to store LetsEncrypt certs(if addr's port is :443)           | /app/certs        |
| DERP_CERT_MODE      | false    | mode for getting a cert. possible options: manual, letsencrypt         | letsencrypt       |
| DERP_ADDR           | false    | listening server address                                               | :443              |
| DERP_STUN           | false    | also run a STUN server                                                 | true              |
| DERP_HTTP_PORT      | false    | The port on which to serve HTTP. Set to -1 to disable                  | 80                |
| DERP_VERIFY_CLIENTS | false    | verify clients to this DERP server through a local tailscaled instance | false             |

# Usage

Fully DERP setup offical documentation: https://tailscale.com/kb/1118/custom-derp-servers/

```yml
version: "3"
services:
  derp:
    image: superng6/derper
    ports:
      - 3478:3478/udp
    environment:
      - DERP_DOMAIN=derper.your-domain.com
      - DERP_ADDR=:443
      - DERP_CERT_DIR=/app/certs
      - DERP_VERIFY_CLIENTS=true
      - DERP_HTTP_PORT=-1
    volumes:
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
      - ./tailscale/tailscale:/var/run/tailscale
      - ./certs:/app/certs
    restart: unless-stopped  
    depends_on:
      - tailscale

  tailscale:
    hostname: tailscale-ten
    image: tailscale/tailscale:stable
    volumes:
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro
      - /dev/net/tun:/dev/net/tun # Required for tailscale to work
      - ./tailscale/tailscale:/var/run/tailscale
      - ./tailscale/lib:/var/lib # State data will be stored in this directory
    cap_add:
      # Required for tailscale to work
      - NET_ADMIN
      - NET_RAW
    command: tailscaled
    restart: unless-stopped
```