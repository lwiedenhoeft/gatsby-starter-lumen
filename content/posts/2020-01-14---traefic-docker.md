---
title: Traefik - Docker Container with Lets Encrypt
date: "2020-01-04T20:20:32.669Z"
template: "post"
draft: false
slug: "traefic-docker-encrypt"
category: "Container"
tags:
  - "Container"
description: "Traefik is a Docker-aware reverse proxy that ships with its own monitoring dashboard. In this post, I will show you how you can use Traefik to route requests to a web application container."
socialImage: "/media/42-line-bible.jpg"
---
## Introduction

Traefik is a Docker-aware reverse proxy that ships with its own monitoring dashboard. In this post, I will show you how you can use Traefik to route requests to a web application container. You will configure Traefik to serve everything over HTTPS using Let’s Encrypt.

Please note that I won't explain in detail what Traefik is since it may needs his own article and I will focus on the deployment and configuration. This example will only work with Traefik v2.

## Traefik - A Story of Labels & Containers

![alt text][logo]

[logo]: https://docs.traefik.io/assets/img/providers/docker.png "Traefik docker provider"

We use the Traefik Docker Provider. In that mode, Traefik will watches for container labels on the docker engine.
When using Docker Compose, labels are specified by the directive labels from the "services" objects.

```yaml
version: "3"
services:
  my-container:
    # ...
    labels:
      - traefik.http.routers.my-container.rule=Host(`mydomain.com`)
```

## Basic setup

Below a very basic setup for Traefik with some explanations.

```yaml
version: "3.5"
services:
  traefik:
    image: "traefik:latest"
    container_name: "traefik"
    restart: always
    security_opt:
      - no-new-privileges:true
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/opt/traefik/:/opt/traefik/"  
```

Basic security. Disable container processes from gaining new privileges.

```yaml
- no-new-privileges:true
```

Enable the docker provider.

```yaml
- "--providers.docker=true"
```

Tell Traefik to not expose container by default: only expose container that are explicitly enabled (using label traefik.enabled).

```yaml
- "--providers.docker.exposedbydefault=false"
```

Create an entrypoint named web exposed on port 80. This entrypoint will be used to indicate on which port your application should be exposed.

```yaml
- "--entrypoints.web.address=:80"
```

## Lets Encrypt Setup

In order to enable the Lets Encrypt certificate challenge, we have to add some additional command parameters.

```yaml
version: "3.5"
services:
  traefik:
    image: "traefik:latest"
    container_name: "traefik"
    restart: always
    security_opt:
      - no-new-privileges:true
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesResolvers.mytlschallenge.acme.httpChallenge=true"
      - "--certificatesResolvers.mytlschallenge.acme.httpChallenge.entryPoint=web"
        # - "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.mytlschallenge.acme.email=example@mail.com"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/opt/traefik/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "/opt/traefik/:/opt/traefik/"  
```

Create an entrypoint named websecure exposed on port 443 (https). This entrypoint will be used to indicate on which port your application should be exposed.

```yaml
- "--entrypoints.websecure.address=:443"
```

Tell Traefik that the challenge named mytlschallenge will use the http challenge.

```yaml
- "--certificatesResolvers.mytlschallenge.acme.httpChallenge=true"
```

We are going to expose the http challenge over http (logic since we have no certificate yet)

```yaml
- "--certificatesResolvers.mytlschallenge.acme.httpChallenge.entryPoint=web"
```

Set your CA-Server. Helpfull if you want to use the Let's Encrypt staging server

```yaml
- "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
# default -> - "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-v02.api.letsencrypt.org/directory"
```

The email used to create a letsencrypt account

```yaml
- "--certificatesresolvers.mytlschallenge.acme.email=example@mail.com"
```

Define where Traefik should persist the certificates.

```yaml
- "--certificatesresolvers.mytlschallenge.acme.storage=/opt/traefik/acme.json"
```

## Lets encrypt with one web app

Next step, we will add an example app and let the app serve via https.

```yaml
version: "3.5"
services:
  traefik:
    image: "traefik:latest"
    container_name: "traefik"
    restart: always
    security_opt:
      - no-new-privileges:true
    command:
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesResolvers.mytlschallenge.acme.httpChallenge=true"
      - "--certificatesResolvers.mytlschallenge.acme.httpChallenge.entryPoint=web"
        # - "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.mytlschallenge.acme.caserver=https://acme-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.mytlschallenge.acme.email=example@mail.com"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/opt/traefik/acme.json"
  example:
    image: "containous/whoami"
    restart: always
    security_opt:
      - no-new-privileges:true
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.example.rule=Host(`www.example.org`)||Host(`example.org`)"
      - "traefik.http.routers.example.entrypoints=websecure"
      - "traefik.http.routers.example.tls.certresolver=mytlschallenge"
      - "traefik.http.middlewares.example.compress=true"
      - "traefik.http.middlewares.exampleHeader.headers.sslredirect=true"
    depends_on:
      - traefik
```

This flag tells Traefik to expose the container, needed because we explicitly disable container auto exposition with providers.docker.exposedbydefault.

```yaml
- "traefik.enable=true"
```

Create a Host Matching rule. Here we tell Traefik to redirect all traffic coming with host localhost to this container.

```yaml
- "traefik.http.routers.example.rule=Host(`www.larstogo.de`)||Host(`larstogo.de`)"
```

Tells Traefik that this container will be exposed using the websecure entrypoint.

```yaml
- "traefik.http.routers.example.entrypoints=websecure"
```

To specify which certificate resolver we wanna use. Here we are using the mytlschallenge declared before.

```yaml
- "traefik.http.routers.example.tls.certresolver=mytlschallenge"
```

The Compress middleware enables the gzip compression.
https://docs.traefik.io/middlewares/compress/

```yaml
- "traefik.http.middlewares.example.compress=true"
```

Enable the ssl redict.

```yaml
- "traefik.http.middlewares.exampleHeader.headers.sslredirect=true"
```

## Additional Links

* Based on https://creekorful.me/how-to-install-traefik-2-docker-swarm/
* https://docs.traefik.io/providers/docker/
* https://www.digitalocean.com/community/tutorials/how-to-use-traefik-as-a-reverse-proxy-for-docker-containers-on-debian-9