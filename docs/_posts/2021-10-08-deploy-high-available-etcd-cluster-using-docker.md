---
layout: post
title: Deploy a high available etcd cluster using docker
description: A very simple way to deploy an etcd cluster using docker, docker-compose or using a docker swarm cluster
categories: [Docker,Docker Swarm,Etcd,Nginx,Nginx Proxy]
---

![etcd](https://garutilorenzo.github.io/images/etcd.png)

[etcd](https://etcd.io/) is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines. It gracefully handles leader elections during network partitions and can tolerate machine failure, even in the leader node.

In this post we will see how to deploy an etcd cluster using docker and docker compose.

## Prerequisites

* docker
* docker-compose
* docker swarm cluster (optional)

Additional requirements for testing purposes:

* python3
* pipenv
* pip3

## Configuration overview

### Nginx

The nginx configuration is very simple. We need to create one upstream section and declare our etcd server names.

```
    upstream etcd_servers {
        least_conn;
        server etcd-00:2379 max_fails=3 fail_timeout=5s;
        server etcd-01:2379 max_fails=3 fail_timeout=5s;
        server etcd-02:2379 max_fails=3 fail_timeout=5s;
    }
```

**NOTE**: Since we are running on docker the server names are resolved by the internal dns. The name is the name of our services declared in docker-compose.yml (etcd-00, etcd-01, etcd-02)


now we need to declare the server section, with the listen port (etcd default listen port) and the proxy_pass rule which will route the traffic to our etcd services:

```
    server {
        listen     2379;
        proxy_pass etcd_servers;
    }
```

the final configuration will look like:

```
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

stream {
    log_format  basic   '$time_iso8601 $remote_addr '
                        '$protocol $status $bytes_sent $bytes_received '
                        '$session_time $upstream_addr '
                        '"$upstream_bytes_sent" "$upstream_bytes_received" "$upstream_connect_time"';

    access_log  /dev/stdout basic;

    upstream etcd_servers {
        least_conn;
        server etcd-00:2379 max_fails=3 fail_timeout=5s;
        server etcd-01:2379 max_fails=3 fail_timeout=5s;
        server etcd-02:2379 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     2379;
        proxy_pass etcd_servers;
    }
}
```

### Etcd

To configure etcd in [cluster](https://etcd.io/docs/v3.5/op-guide/clustering/) mode, on each container we need to specify the following settings:

```
    command:
      - etcd
      - --name=etcd-02
      - --data-dir=data.etcd
      - --advertise-client-urls=http://etcd-02:2379
      - --listen-client-urls=http://0.0.0.0:2379
      - --initial-advertise-peer-urls=http://etcd-02:2380
      - --listen-peer-urls=http://0.0.0.0:2380
      - --initial-cluster=etcd-00=http://etcd-00:2380,etcd-01=http://etcd-01:2380,etcd-02=http://etcd-02:2380
      - --initial-cluster-state=new
      - --initial-cluster-token=etcd-cluster-1
```

where:

* --name: Human-readable name for this member.
* --data-dir: Path to the data directory.
* --advertise-client-urls: List of this member’s client URLs to advertise to the rest of the cluster. These URLs can contain domain names.
* --listen-client-urls: List of URLs to listen on for client traffic. This flag tells the etcd to accept incoming requests from the clients on the specified scheme://IP:port combinations. 
* --initial-advertise-peer-urls: List of this member’s peer URLs to advertise to the rest of the cluster. These addresses are used for communicating etcd data around the cluster. At least one must be routable to all cluster members. These URLs can contain domain names.
* --listen-peer-urls: List of URLs to listen on for peer traffic. This flag tells the etcd to accept incoming requests from its peers on the specified scheme://IP:port combinations. 
* --initial-cluster: Initial cluster configuration for bootstrapping.
* --initial-cluster-state: Initial cluster state (“new” or “existing”). Set to new for all members present during initial static or DNS bootstrapping. If this option is set to existing, etcd will attempt to join the existing cluster. If the wrong value is set, etcd will attempt to start but fail safely.
* --initial-cluster-token: Initial cluster token for the etcd cluster during bootstrap.

All the configuration flags are available [here](https://etcd.io/docs/v3.5/op-guide/configuration/)

## Deploy etcd cluster with docker compose

The first step is to clone [this](https://github.com/garutilorenzo/docker-etcd-cluster.git) repository:

```
git clone https://github.com/garutilorenzo/docker-etcd-cluster.git
```

then enter in the repo directory an bring up the environment:

```
cd docker-etcd-cluster 
docker-compose up -d
```

now check the status of the environment and wait for the containers to be ready:

```
docker-compose ps


     Name                   Command               State                        Ports                      
----------------------------------------------------------------------------------------------------------
etcd_etcd-00_1   etcd --name=etcd-00 --data ...   Up      2379/tcp, 2380/tcp                              
etcd_etcd-01_1   etcd --name=etcd-01 --data ...   Up      2379/tcp, 2380/tcp                              
etcd_etcd-02_1   etcd --name=etcd-02 --data ...   Up      2379/tcp, 2380/tcp                              
etcd_nginx_1     /docker-entrypoint.sh ngin ...   Up      0.0.0.0:2379->2379/tcp,:::2379->2379/tcp, 80/tcp
```

## Test the environment

To test the environment you need [pipenv](https://pipenv.pypa.io/en/latest/install/#installing-pipenv) installed.
Once you have pipenv installed:

```
pipenv shell
pip install -r requirements.txt
python test/etcd-test.py 
hey key1
hey key2
```

Check the log of the nginx service to see the traffic redirected to the etcd hosts:

```
docker-compose logs -f nginx

Attaching to etcd_nginx_1
nginx_1    | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
nginx_1    | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
nginx_1    | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
nginx_1    | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
nginx_1    | 10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
nginx_1    | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
nginx_1    | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
nginx_1    | /docker-entrypoint.sh: Configuration complete; ready for start up
nginx_1    | 2021-10-08T09:41:23+00:00 172.28.0.1 TCP 200 422 665 0.052 172.28.0.3:2379 "665" "422" "0.000"
nginx_1    | 2021-10-08T09:41:24+00:00 172.28.0.1 TCP 200 422 665 0.046 172.28.0.2:2379 "665" "422" "0.000"
nginx_1    | 2021-10-08T09:50:56+00:00 172.28.0.1 TCP 200 422 665 0.029 172.28.0.4:2379 "665" "422" "0.000"
```


## Docker swarm stack

To deploy the etcd cluster on a docker [swarm](https://docs.docker.com/engine/swarm/) cluster:

```
docker stack deploy -c etcd-stack.yml etcd
```

Check the status of the deployment:

```
docker stack ps etcd

mx6fvfwye547   etcd_etcd-00.1       quay.io/coreos/etcd:v3.5.0   node-2    Running         Running 3 hours ago                                        
wybd7n4oitae   etcd_etcd-01.1       quay.io/coreos/etcd:v3.5.0   node-4    Running         Running 3 hours ago                                        
rmlycc3uvc8t   etcd_etcd-02.1       quay.io/coreos/etcd:v3.5.0   node-2    Running         Running 3 hours ago                                        
rexh1smoalpo   etcd_nginx.1         nginx:alpine                 node-2    Running         Running 21 hours ago    

docker service ls
ID             NAME                  MODE         REPLICAS   IMAGE                          PORTS
1u709kzgmo2b   etcd_etcd-00          replicated   1/1        quay.io/coreos/etcd:v3.5.0     
m7ze76xi58ww   etcd_etcd-01          replicated   1/1        quay.io/coreos/etcd:v3.5.0     
1535r562g3az   etcd_etcd-02          replicated   1/1        quay.io/coreos/etcd:v3.5.0     
v8n8qlo3dm30   etcd_nginx            replicated   1/1        nginx:alpine                   *:2379->2379/tcp
```

If you would like to test the swarm setup, open test/etcd-test.py and change *127.0.0.1* with one ip of a server in your docker swarm cluster (or the ip of your LB) and run the test.