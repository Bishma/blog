---
title: "MrHomn: Migration to Docker"
date: 2021-07-18T14:55:16-07:00
description: "Proxy, Home Assistant and DB's. Oh my."
images: []
tags: [ "Summer of 42", "sysadmin", "mrhomn", "home assistant", "docker" ]
draft: false
---

# {{< param description >}}

[Last time]({{< ref "/posts/mrhomn" >}} "MrHomn") on the MrHome project I talked about wanting to expand my home automation system and deciding to do it via docker-compose. This is mainly because I wanted to learn more about deploying containerized apps using naked docker-compose. The other reason is that I have a lot of spare compute and IO resources on my NAS and docker was the most direct approach to running [home assistant](https://www.home-assistant.io/) there.

{{< figure src="/images/mrhomn-migration-to-docker/mrhomn1.png" width="70%" >}}{{< /figure >}}

## Goals

The first step to expanding is getting my existing (and extensive) system migrated from hass os to docker. This means having:

* A proxy / router that can serve different containers on different paths
* TLS Certs (LetsEncrypt)
* Home Assistant
* A sql database
* AppDaemon
* Z-Wave JS

I need to route incoming traffic between nodes in my docker stack via path. My Alexa skill has to be publicly served on port 443 and I don't want to have a second endpoint running, so I am implementing a path based router. I also wanted to handle my TLS cert termination at this layer as I have no need to pass encrypted traffic around inside my docker network.

I started, as one does, with [Nginx](https://hub.docker.com/_/nginx):alpine + [Certbot](https://hub.docker.com/r/certbot/certbot) in seperate containers and it worked fine, but I didn't like needing to handle certs "myself." That certbot container is just one more breakpoint. I thought about going with [nginx proxy manager](https://nginxproxymanager.com/) because I've worked with it before and I know it's dead simple to stand up in docker... but also I've used it before and this is meant to be a learning experience. So I ended up going with [Traefik](https://doc.traefik.io/traefik/).

## Compose file setup

My full [compose file](https://github.com/Bishma/MrHomn/blob/main/docker-compose.yaml) is available for review. I'll do my best to explain each service here. This section is going to be dense but hopefully it save someone else some of my debugging time.

I'll go over each of my starting service definitions and attempt keep line numbers consistent.

### Traefik

Starting with the container basics. I'm tagging the current minor version of traefik from docker hub.

When I was developing I set the log level to debug, but for normal operations I think warning is sufficient. I'm also turning on access logs and having them just go to stdout by not specifying an output file.

```yaml {linenos=table,linenostart=3}
traefik:
    image: "traefik:v2.5"
    container_name: "traefik"
    command:
      - "--log.level=WARN"
      - "--api.insecure=true"
      - "--accesslog=true"
```

We need to set up the provider now. Obviously we're using docker. I'll be configuring the routers using docker compose labels, so the set up here is pretty simple.

```yaml {linenos=table,linenostart=10}
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
```

Now I want to redirect all port 80 traffic to port 443 via permanent redirect.

```yaml {linenos=table,linenostart=12}
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entrypoint.permanent=true"
```

Now we get to the entry point that matters. This "websecure" endpoint will handle traffic for both Home Assistant and AppDaemon.

```yaml {linenos=table,linenostart=16}
      - "--entrypoints.websecure.address=:443"
      - "--entrypoints.websecure.forwardedHeaders.insecure"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=developer@dereksrose.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
```

The host ports here (8321 and 8322) are here translated back into ports 80 and 443 respectively. I also open up the port to Traefik's UI.

```yaml {linenos=table,linenostart=21}
    ports:
      - "8321:80"
      - "8322:443"
      - "8080:8080"
```

In order to get things running I'm just going to set up logging as JSON and a short retention. I'll get into more involved logging later... probably.

```yaml {linenos=table,linenostart=25}
    volumes:
      - "./letsencrypt:/letsencrypt"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    logging:
      driver: json-file
      options:
        max-size: "10m"
```

### Home Assistant

Pretty standard stuff. The important stuff for routing is down at the bottom.

```yaml {linenos=table,linenostart=32}
hass:
    container_name: homeassistant
    image: homeassistant/home-assistant:2021.7
    restart: unless-stopped
    environment:
      - TZ=${TZ}
    expose:
      - 8123
    ports:
      - "8123:8123"
    volumes:
      - ./hass:/config
    logging:
      driver: json-file
      options:
        max-size: "10m"
    depends_on:
      - sql_db
```

First we need to tell Traefik to enable routing to this container, since we turned automatic routing off. Then we're going to define the rules for the websecure endpoint to route to Home Assistant. And we need to tell it to use the Let's Encrypt cert resolver.

```yaml {linenos=table,linenostart=50}
    labels:
      - traefik.enable=true
      - traefik.http.routers.homeassistant.rule=Host(`my.site.com`)
      - traefik.http.routers.homeassistant.entrypoints=websecure
      - traefik.http.routers.homeassistant.tls.certresolver=myresolver
      - traefik.http.services.homeassistant.loadbalancer.server.port=8123
```

### AppDaemon

The other docker service that needs to make use of Traefik's websecure endpoint is [AppDaemon](https://appdaemon.readthedocs.io/en/latest/). At the moment I'm only using AppDaemon for my Alexa skill but I plan to make more APIs and other projects that can make use of the coupling to Home Assistant.

```yaml {linenos=table,linenostart=56}
appdaemon:
    container_name: appdaemon
    restart: unless-stopped
    image: acockburn/appdaemon:4.0.8
    ports:
      - "5050:5050"
    expose:
      - 5050
    volumes:
      - ./appdaemon/config:/conf
    logging:
      driver: json-file
      options:
        max-size: "10m"
    depends_on:
      - hass
```

The only difference there, compared to home assistant, is that our routing rule requires both my domain and an API path. This is the patch that Alexa is pointed at.

```yaml {linenos=table,linenostart=72}
    labels:
      - traefik.enable=true
      - traefik.http.routers.appdaemon.rule=(Host(`my.site.com`) && PathPrefix(`/api/appdaemon/`))
      - traefik.http.routers.appdaemon.entrypoints=websecure
      - traefik.http.routers.appdaemon.tls.certresolver=myresolver
      - traefik.http.services.appdaemon.loadbalancer.server.port=5050
```

### Z-Wave JS

In order to give the Home Assistant Z Wave JS integration a local z wave server to talk to, I'm using the zwave2mqtt image. For the time being I have mqtt turned off (via the UI on port 8080), but that will change soon enough.

```yaml {linenos=table,linenostart=78}
zwavejs:
    container_name: zwavejs
    image: zwavejs/zwavejs2mqtt
    restart: unless-stopped
    environment:
      - TZ=${TZ}
    ports:
      - "3000:3000"
      - "8091:8091"
    tty: true
    volumes:
      - ./zwavejs:/usr/src/app/store
    devices: 
      - "/dev/ttyACM0:/dev/ttyACM0"
    logging:
      driver: json-file
      options:
        max-size: "10m"
```

I've given this service access to my Synology's front USB port since I have a [Aeotec z-wave usb stick](https://aeotec.com/z-wave-usb-stick/) there.

### SQL

I've gone with MariaDB for SQL. This currently only drives home assistant, but I will soon be restoring some Alexa skill functionality that also requires SQL.

```yaml {linenos=table,linenostart=96}
  sql_db:
    container_name: sql_db
    image: linuxserver/mariadb # https://hub.docker.com/r/linuxserver/mariadb
    restart: always
    environment:
      - PUID="${UID}"
      - PGID="${GID}"
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT}
      - TZ=${TZ}
      - MYSQL_DATABASE=hass
      - MYSQL_USER=hass
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    expose:
      - 3306
    volumes:
      - ./sql_db/config:/config
    logging:
      driver: json-file
      options:
        max-size: "10m"
```

## Synology IP Tables Fix

To be honest I don't know enough about networking to know who's ultimately at fault here, but the default interaction between synology and docker keep bridge networks from being able to see real end user IPs. I don't know how many hours I spend messing with proxy protocol settings in nginx and traefik.

The fix that finally worked was altering my synology's iptables settings. I don't know what good karma finally led me to [pedrolamas/docker-iptables-fix.sh](https://gist.github.com/pedrolamas/db809a2b9112166da4a2dbf8e3a72ae9), but it was a turning point for this project.

## The port dance

For the sake of compatibility I need Traefik to listen for traffic on public  ports 80 and 443 but I'm already using these ports (privately) on my Synology Diskstation for internal traffic routing through nginx. So traffic will hit my my WAF on those ports and then forwards to ports 8321 and 8322 on my Synology.

To keep it all straight I made a flow chart.

{{< figure src="/images/mrhomn-migration-to-docker/traefik_port_dance.png" >}}
A diagram of the port flow of incoming web requests.
{{< /figure >}}

BTW, {{< lightbox-link image="/images/mrhomn-migration-to-docker/nginx_port_dance.png" caption="All the routing that nginx needed to also handle certbot." >}}this is what it looked like{{< /lightbox-link >}} when I first set it up with Nginx. I don't know why, but having to maintain that extra route for certbot was a, if you'll forgive the idiom, a bridge too far.

# The migration process

Now that I'm routing I can take down my existing [Home Assistant OS](https://www.home-assistant.io/installation/) based install and move it over to my docker setup.

## Just the foundation

I ultimately went with a semi start-over approach. I copied all my yaml files (`configuration.yaml`, `scripts.yaml`, `secrets.yaml`, etc) and the lovelace json files from inside `.storage`. I deleted everything that was generated by home assistant as I was working out my routing system and put the copied files in their place. Then I restarted home-assistant and manually set up all UI based integrations. This took minutes, and other that needing to clear browser cache and re-setup our mobile apps it was pretty painless.

## Updates

My Home Assistant doesn't use their handy [default_config](https://www.home-assistant.io/integrations/default_config/) because I don't like having autodiscovery turned on. The downside to doing things manually is that I miss out on new developments that are normally added to this `default_config` alias.

So I went in and added some of the things I've missed out on since I last did a major update including [my](https://www.home-assistant.io/integrations/my/) and [stream](https://www.home-assistant.io/integrations/stream/). Then I cleaned up some other messes here and there as well.

# Aftermath

Very minor things were broken by my brute force migration.

I ran into 2 devices whose names had changed at somepoint (on the hardware) but which had their old names cached in my old install. I had to go into one script, one automation, and a couple lovelace cards to update things to the correct (newer) name.

Because I didn't copy over anything from auth we had to clear our browser caches and re-setup our mobile apps.

# Coming next

I'm currently writing a very basic DR script in python. Honestly I'm doing it manly to play with Github Copilot. Then I think I want to start playing with [Influxdb and Grafana](https://grafana.com/docs/grafana/latest/datasources/influxdb/) to visualize some of my home assistant data.

Hopefully I don't run out of RAM, an upgrade isn't in the budget.