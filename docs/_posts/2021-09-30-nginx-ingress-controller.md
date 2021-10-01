---
layout: post
title: Docker swarm nginx ingress controller
description: A simple way for routing external requests inside a docker swarm cluster
categories: [Docker,Docker Swarm,Nginx,Ingress Controller]
---

In this tutorial we are going to see how to use Nginx as an ingress controller for our Docker swarm cluster.

![nginx-ingress-controller-small](https://garutilorenzo.github.io/images/nginx-ingress-controller-small.png)


## Prerequisites

For this tutorial we only need a running Docker swarm cluster. You can setup a docker swam cluster in a [few setp](https://docs.docker.com/engine/swarm/).

## Nginx ingress controller

The image that we will use is based on the excellent work of [foxylion](https://github.com/foxylion) then revisited on my [docker swarm ingress](https://github.com/garutilorenzo/docker-swarm-ingress) image.

The image it’s very simple, the magic part is the ingress.py file so let’s take a look on how the magic happens.

The image needs to be deployed on a manager node, when the image starts the docker-entrypoint.sh is called:

```
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 \      -subj "/C=IT/ST=Denial/L=Italy/O=IT/CN=dummy.cert.io" \      -keyout /etc/nginx/default.key  -out /etc/nginx/default.crtpython ingress.py &
echo $! > /ingress/ingress.pid
exec nginx -g "daemon off;"
```

In the above snippet we see the main part of the docker-entrypoint.sh.

In our container there are now two process:

* the ingress.py “daemon”
* and the nginx process

The first line of our snippet create a dummy ssl certificate for nginx, we will cover this in the “SSL mode in depth”.

The ingress.py daemon then will check all the services on the docker swarm cluster. If a service has a label named “ingress.host” the nginx configuration will be regenerated based on the value of the labels.

Here is a list of valid labels:

* ingress.host: the nginx server name (virtualhost). Example: my-service.company.tld
* ingress.port: the port which serves the service in the cluster.
* ingress.virtual_proto: the protocol used to connect to the backends
* ingress.ssl: specify enable to enable ssl-assthrough. More detail on ssl in the “SSL mode in depth”
* ingress.ssl_redirect: specify enable to enable http to https redirect

There are also some environment variales that we can specify on our nginx ingress controller container. Here the list of accepted environment variables:

* DOCKER_HOST: the connect string where docker-py library tries to connect to the docker daemon
* UPDATE_INTERVAL: the time in seconds that ingress.py wait before checking for new services in the docker swarm cluster. Default 30 seconds.
* Debug: enable or disable debug mode
* USE_REQUEST_ID: enable or disable Request-Id header
* LOG_FORMAT: specify log format, valid values are json, custom or default
* LOG_CUSTOM: specify the nginx log format
* PROXY_MODE: define nginx SSL proxy mode. Valid values are ssl-passthrough (default) or an used defined value to enable SSL bridging or SSL termination mode. More detail on ssl in the “SSL mode in depth”

## SSL mode in depth

### SSL Passthrough (default)

![ssl-passthrough](https://garutilorenzo.github.io/images/ssl-passthrough.png)

SSL passthrough passes HTTPS traffic to a backend server without decrypting the traffic on the load balancer. The data passes through fully encrypted, which precludes any layer 7 actions.

### SSL Termination/Offloloading

![ssl-offloading](https://garutilorenzo.github.io/images/ssl-termination.png)

SSL termination (a.k.a. SSL Offloading) decrypts all HTTPS traffics when it arrives at the load balancer (our docker swarm ingress controller), and the data is sent to the destination server as plain HTTP traffic (our http backend deployment)

### SSL Bridging

![ssl-bridging](https://garutilorenzo.github.io/images/ssl-bridging.png)

Opposed to SSL termination the traffic from the load balancer and the destination is not in plain HTTP traffic but the traffic is encrypted again.

## Setup the stack

We are now ready to deploy our nginx ingress controller, in one docker swarm manager node download my docker swarm ingress repository:

```
git clone https://github.com/garutilorenzo/docker-swarm-ingress.git
```

we need to create an overlay network first. This network will then be attached to our backends. Create the network with:

```
docker network create --driver overlay ingress-routing
```

then deploy the stack:

```
cd docker-swarm-ingress/
docker stack deploy -c examples/docker-ingress-stack.yml nginx-ingress
```

we can check the status of our stack using docker stack ps:

```
docker stack ps nginx-ingress

ID             NAME                    IMAGE                                       NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
x47oswdzbnlp   nginx-ingress_nginx.1   garutilorenzo/docker-swarm-ingress:latest   node-2    Running         Running 12 seconds ago
```

we can also check if our nginx ingress controller is working correctly using curl or a web browser.

![swarm-ingress](https://garutilorenzo.github.io/images/swarm-ingress.png)

We are now ready to deploy our first backend service, to do this we can use an example service:

```
docker stack deploy -c examples/example-service.yml example-http
```

if we check the nginx ingress controller log we can see that the nginx server has restarted:

```
docker service logs -f --tail=100 nginx-ingress_nginx

nginx-ingress_nginx.1.njwey0rlfvdw@node-2    | 2021/09/30 14:05:12 [notice] 11#11: exiting
nginx-ingress_nginx.1.njwey0rlfvdw@node-2    | 2021/09/30 14:05:12 [notice] 11#11: exit
nginx-ingress_nginx.1.njwey0rlfvdw@node-2    | 2021/09/30 14:05:42 [notice] 41#41: signal process started
```

and now we can try to reach my-service.company.tld (update your DNS or your host file and set my-service.company.tld pointing to one of your docker swarm cluster ip’s)

![swarm-http-backend](https://garutilorenzo.github.io/images/http-backend.png)

### HTTPS, SSL passthrough mode

We can now try to expose my-service.company.tld using https with SSL passthrough mode.

First we need to delete our example-http stack:

```
docker stack rm example-http
```

then we need to create our self-signed ssl certificates and we need also to create the nginx secrets:

```
mkdir certs && sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./certs/nginx.key -out ./certs/nginx.crtGenerating a RSA private key
...................................................................................................+++++
.....................+++++
writing new private key to './certs/nginx.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:IT
State or Province Name (full name) [Some-State]:Italy
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:my-service.company.tld
Email Address []:me@company.tld

docker secret create nginx_cert certs/nginx.crt
docker secret create nginx_key certs/nginx.key
```

now we can deploy our ssl backends:

```
docker stack deploy -c examples/example-ssl-service.yml example-httpsCreating config example-https_nginx_config

Creating config example-https_nginx_options
Creating config example-https_nginx_dhparams
Creating service example-https_nginx
```

check the nginx ingress controller logs and when the nginx daemon is restarted try to reach my-service.company.tld:

![swarm-https-backend](https://garutilorenzo.github.io/images/https-backend.png)

we are now exposing our example-https using SSL passthrough.

### HTTPS, SSL Termination/Offloloading

We can now test SSL Termination/Offloading mode, we need to delte both the stack:

```
docker stack rm example-https

Removing service example-https_nginx
Removing config example-https_nginx_dhparams
Removing config example-https_nginx_options
Removing config example-https_nginx_config

docker stack rm nginx-ingress
Removing service nginx-ingress_nginx
```

Now we need to create the ssl certificates for our domain, exposed directly by the nginx ingress controller:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./certs/my-service.key -out ./certs/my-service.crt
Generating a RSA private key
......+++++
...............................................................+++++
writing new private key to './certs/my-service.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:IT
State or Province Name (full name) [Some-State]:Italy
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:my-service.company.tld

docker secret create my-service.company.tld.key certs/my-service.key
docker secret create my-service.company.tld.crt certs/my-service.crt
```

Now we can deploy our nginx ingress controller in SSL Termination/Offloloading mode.

```
docker stack deploy -c examples/docker-ingress-stack-ssl_term_bridg.yml nginx-ingress
```

and we can also deploy our http example backend:

```
docker stack deploy -c examples/example-service-ssl-termination.yml example-https-termination
```

check the nginx ingress controller logs and when the nginx daemon is restarted try to reach my-service.company.tld. We are now reacing my-service.company.tld in https mode but unlike the SSL passthrough mode the ssl certificates are served directly by our nginx ingress controller.

### HTTPS, SSL Bridging

For the SSL bridging test we need to delete only our backend service “example-https-termination”. The configuration of the nginx ingress controller for SSL bridging and SSL Termination/Offloloading are the same.

```
docker stack rm example-https-terminationdocker stack deploy -c examples/example-service-ssl-bridging.yml example-htts-bridging
```

check the nginx ingress controller logs and when the nginx daemon is restarted try to reach my-service.company.tld. We are now reacing my-service.company.tld in https mode the ssl certificates are served directly by our nginx ingress controller and the communication between the nginx ingress controller and our backend service “example-htts-bridging” is encrypted too.

### Nginx tips

For the SSL mode nginx is acting as a layer 7 and layer 4 porxy, the nginx configurations to achieve this is the following:

```
stream {
    map $ssl_preread_server_name $name {
        my-service.company.tld       backend-example-https_nginx;
        
    }
  
    # my-service.company.tld - ywy8bs1b6h8td118mkjrr70eq - HTTPS Passthrough
    upstream backend-example-https_nginx {
        server example-https_nginx:443;
    }
    
    proxy_protocol on;
  
    server {
        listen      443;
        proxy_pass  $name;
        ssl_preread on;
    }
}
```

and in the backend configuration we need to add this to the nginx config:

```
listen 443 ssl proxy_protocol;         
set_real_ip_from nginx-ingress_nginx; # <- name of our nginx ingress controller service
real_ip_header proxy_protocol;
```