# ODISSEI Stack
This repository aims at providing, for hosting Dataverse, both of a local dev stack and a SSL enabled production stack with FQDN. 

Both of the stacks have 2 parts:
 * yaml file for proxy
 * and yaml file for other services

The purpose of splitting the proxy part from the rest of the services is to keep the services decoupled so each of them 
can be safely restarted without disrupt the rest. Restart the proxy will cut the connectivity, however, restarting the 
proxy would take 1 or 2 seconds only, so the downtime is manageable.

One other purpose of using proxy is to provide secured connectivity to other ODISSEI related services. The proxy will 
not only handle SSL certificate but also handle multi subdomain scenario. While 
there are more services
### Preparation
Before you can spin up any stack, you have to create the public facing network for the proxy
```shell
docker network create traefik-public
```

### Building dev stack
This stack is meant to be used mainly as the dev stack on local host. 
It can be run on any remote server as well, i.e for demo purpose, when `HOSTNAME` in associated `env` file is 
properly set. 
#### Step 1: Start the proxy

```shell
docker-compose --env-file .env.dev -f docker-compose-no-ssl-proxy.yaml up -d
```

#### Step 2: Start the services
```shell
docker-compose --env-file .env.dev -f docker-compose-no-ssl-service.yaml up -d
```
As specified in the `docker-compose-no-ssl.env`, if you visit `http://dev.localhost` on your local machine now, you will 
see the dataverse instance. 

 * `solr.dev.localhost` will take you to the solr page
 * `dbgui.dev.localhost` will take you to the db admin page
 * `dev.localhost:8085` will take you to the demo contact page for CBS

#### Step 3: Edit /etc/hosts
Add the following entry to `/etc/hosts` to enable routing to localhost
```shell
127.0.0.1 dev.localhost
```

### Building Production (or public test) stack
To build the public facing services, though not necessarily 'public', one can utilize the following procedure.

#### Step 1: Start the proxy 

```shell
docker-compose --env-file .env.prod -f docker-compose-ssl-proxy.yaml up -d
```

#### Step 2: Start the services
```shell
docker-compose --env-file .env.prod -f docker-compose-ssl-service.yaml up -d
```
As specified in the `.env.prod`, if you visit `https://portal.odissei.nl`, you will see the dataverse instance.

Other available instances:

* `solr.odissei.nl` - solr page
* `dbgui.odissei.nl` - db admin page
* `cbs-contact.odissei.nl` - the demo contact page for CBS

Slight modification, change the hostname variable used in all the yaml files, one public demo instance can be 
easily built. For example, `HOSTNAME=test.odissei.nl` on another VM, test and production stack *cannot* run on the same 
VM, will start an identical instance on subdomain `test.odissei.nl` all the subdomains should be pointed in DNS 
accordingly as well. 
