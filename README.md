# Generate Custom Certificates

This is only to simulate usage of custom certificates and how you load them inside of trafik. 
Traefik generates an ssl-certificate when no custom certificate is provided. 

```shell
mkdir certs
openssl req -newkey rsa:2048 -nodes -keyout certs/cert-app1.key -x509 -days 365 -out certs/cert-app1.crt
openssl req -newkey rsa:2048 -nodes -keyout certs/cert-app2.key -x509 -days 365 -out certs/cert-app2.crt
```

# Deploy it

```
docker network create global-traefik-net --driver=overlay
docker stack deploy -c docker-compose.global.yaml global
ENV=production APP_DOMAIN=app1.localhost STACK_NAME=app1 docker stack deploy -c docker-compose.app1.yaml app1
ENV=production APP_DOMAIN=app2.localhost STACK_NAME=app2 docker stack deploy -c docker-compose.app2.yaml app2
```

# Multiple Environments

Multiple Environments can live side by side as long as the configuration uses the STACK_NAME for unique router and service names.

```
docker network create global-traefik-net --driver=overlay
docker stack deploy -c docker-compose.global.yaml global
ENV=production APP_DOMAIN=app1.localhost STACK_NAME=app1 docker stack deploy -c docker-compose.app1.yaml app1
ENV=staging APP_DOMAIN=app1-staging.localhost STACK_NAME=app1-staging docker stack deploy -c docker-compose.app1.yaml app1-staging
```

# Use Lets Encrypt certificate resolver behind global traefik instance

Multiple Environments can live side by side as long as the configuration uses the STACK_NAME for unique router and service names.

```
docker network create global-traefik-net --driver=overlay
docker stack deploy -c docker-compose.global.yaml global
ENV=production APP_DOMAIN=mydomain STACK_NAME=app docker stack deploy -c docker-compose.app3-letsencrypt.yaml app
ENV=staging APP_DOMAIN=mydomain-staging STACK_NAME=app-staging docker stack deploy -c docker-compose.app3-letsencrypt.yaml app-staging
```

## Let's Encrypt staging certificates

The production server from Let's Encrypt has rate limiting enabled. This means it's easy to run out of certificate requests while trying to setup the configuration for a new application. In those cases the staging server can enabled by adding the following command line parameter:

```shell
- --certificatesresolvers.${STACK_NAME?required}-lets-encrypt-resolver.acme.caServer=https://acme-staging-v02.api.letsencrypt.org/directory
```

A successfully received staging certificate is added to your `acme.json` file and should have an issuer like `FAKE LE INTERMEDIATE X1`.
After switching to the LE production server by commenting out the `caServer` option manuel interference is required. By default the old staging certificate will still be served and no new certificate for production will be requested from LE.
The fake certificate needs to be removed from your `acme.json` file and the traefik instance service for this application needs to be restarted with:

```shell
docker service update app-trafik --force
```

It doesn't seem to be possible to register different certificate resolvers (debug and production) in your traefik configuration. The http challenge seems to interfere with each other leading to a situation where no new certificates can be requested. And because the http challenge has to be performed on port 80, an alternative port cannot be used.

