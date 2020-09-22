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

