# Running Wordpress in Docker

## Basic Setup

Create a file called `.env` in the root of this repository which sets the value of the environment variable placeholders used in `docker-compose.yml`.

Now run: `docker-compose up`

Wordpress will now be running on the host on port 8000.

Navigate to port 8000 on the docker host, and follow the Wordpress setup instructions.  Use the Import tool if you need to do that, and don't forget to set the server host in Wordpress dashboard, otherwise it will populate links to things like stylesheets and images with address `127.0.0.1` which is probably not what you want. 

## Running Wordpress behind a reverse proxy

This is only important if you're running more than one site on the same machine, and each one runs in a separate Apache/Nginx/etc docker container.

The `VIRTUAL_HOST` environment variable is used in `docker-compose.yml` in order to configure the docker-gen ngninx reverse proxy, to host Wordpress behind a subdomain.  See more [https://github.com/jwilder/nginx-proxy](here).  

To start up nginx reverse proxy:

`docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy`

This reverse proxy will handle any inbound requests on port 80 and route them to the appopriate docker container, based on the value of `VIRTUAL_HOST` set on that container.

The reverse proxy is not included in the `docker-compose.yml` file because there is only one of these containers for a number of subdomains, which each might have their own docker-compose configuration.

It may be necessary in future to move to Traefik, as it allows front-end mapping by URL path as well as just subdomain.  A use-case for this would be a project which has front-end code running in one container, and an API server running in another container.  The two containers must naturally publish different ports on the host, yet if the front-end code makes requests to the API server then these two things must run on the same host and port because of the same-origin access policy in javascript.  Traefik solves this problem because it can route traffic to either of the container/ports based on the incoming URL path.  
