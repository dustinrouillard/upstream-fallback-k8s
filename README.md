# Proxy Routing Configurations for DCS

This repository contains the configurations I use to proxy all my web traffic through another server first, which allows me to load balance, upstream fallbacks, ect.

This configuration includes traefik, but could be adapted to use any ingress provider, but make sure your ingress provider can do health checking on it's own, more on why below.

## Exposing External Host as K8S Service

This is not required, but is nice to have internal hostnames for your upstream https service and health service, seen in [proxy/traefik/configmap.yaml](https://github.com/dustinrouillard/upstream-fallback-k8s/tree/master/proxy/traefik/configmap.yaml)

## Health Monitoring

I've configured this using the file configuration method in Traefik to allow me to have health checking, doing the configuration using the ingress resource or traefik ingress resource would prevent me from having health checking to the host, this is because endpoints nor services have health checking on K8S, so we have to do something else.

As configured this will call your health endpoint (configured in [proxy/traefik/configmap.yaml](https://github.com/dustinrouillard/upstream-fallback-k8s/tree/master/proxy/traefik/configmap.yaml))

## TLS Configuration

The SSL certificate I'm using is issued for all the domains I plan on using with the proxy setup, this is important as we use a default ssl certificate in the Traefik config, which is mounted from a kubernetes secret (Seen in [proxy/traefik/deployment.yaml](https://github.com/dustinrouillard/upstream-fallback-k8s/tree/master/proxy/traefik/deployment.yaml) and the config map)

This is super easy to do if you're using cloudflare, with origin certificates.

## Showing the Maintenance Page

This is configured with middlewares to replace the default 502 and 503 http pages with the maintenance service, another middleware is watching the network error rate.

This can be configured to point to basically anything, but I made a maintenance page [here](https://github.com/dustinrouillard/dcs-maintenance-app) and the configrations for deploying this are found in [proxy/maintenance](https://github.com/dustinrouillard/upstream-fallback-k8s/tree/master/proxy/maintenance)

## IP Whitelisting

In order to set this up properly behind cloudflare and before the upstream whitelists should be configured as the provided configs explain, meaning you should whitelist your proxy ip addresses on your upstream server, and you should whitelist the cloudflare ip ranges on your proxy servers, there are middlewares provided to do both of these tasks (provided that you're using Traefik for your upstream and proxy)

## Using this in your own infrastructure

That'll be a bit difficult, but of course it's possible, am around to provide help to anyone who wants to do something similar or needs this for some crucial services, just hit me up on discord, or my twitter, which can both be found on [my website](https://dstn.to)

I'm open-sourcing this because I want to share more of my kubernetes configs, not only for others to learn-from, but for more experienced people to critique.

## How well does this work?

Well, with the limited tests I've done, pretty well.

When your upstream is responding, but it's not handling traffic properly or is refusing the connection it will return the maintenance page instantly.

However when your upstream loses internet or something is causing the connection to hang, it will take about 20 seconds or so to notice and start returning the maintenance page, and about 20 seconds after recovery for it to come back.
