# Barebone Traefik Setup

Goal of this setup was to have https and AWS Route53 intgeration for letsencrypt

## Docker Network

I'm using traefik with an external network, which has to be created before we start working on traefik itself:
`docker network create traefik`

## /etc/default/traefik

Replace the values in `etc_default_traefik` with your own and copy it to /etc/default/traefik:

```
$ sudo cp etc_default_traefik /etc/default/traefik
```

## Letsencrypt

I got most of my info from the [traefik documentation](https://docs.traefik.io/user-guides/docker-compose/acme-dns/).
Here's a list of [supported providers](https://docs.traefik.io/https/acme/#providers) and that's the
[Route53 specific documentation](https://go-acme.github.io/lego/dns/route53/) &ndash; the IAM policy document is
very helpful!

## traefik.toml

TOML is a bit a weird markup language, but you'll get the hang of it quite fast.

```
[certificatesresolvers.example.acme]
  email = "info@example.com"
  storage = "/letsencrypt/acme.json"

  [certificatesresolvers.example.acme.dnsChallenge]
    provider = "route53"
    delayBeforeCheck = 0
```

I moved the configuration of the certificate resolvers to the traefik config file, the only thing you have to change here is
the name of the resolver, e.g.:

```
[certificatesresolvers.mypixelfed.acme]
  email = "info@mypixelfed.com"
  storage = "/letsencrypt/acme.json"

  [certificatesresolvers.mypixelfed.acme.dnsChallenge]
    provider = "route53"
    delayBeforeCheck = 0
```

## pixelfed docker-compose

```
services:
  app:
    image: pixelfed
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik"
      - "traefik.http.routers.pixel.rule=Host(`pixelfed.example.com`)"
      - "traefik.http.routers.pixel.entrypoints=https"
      - "traefik.http.routers.pixel.tls.certresolver=example"
```

What's left to do here is to change the actual domain you're using and the name
of the certificate resolver, which we changed to "mypixelfed" in the above example.
