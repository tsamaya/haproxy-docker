# How to Run HAProxy With Docker

## Init the test env

```bash
docker network create --driver=bridge mynetwork

docker run -d --name web1 --net mynetwork jmalloc/echo-server:latest

docker run -d --name web2 --net mynetwork jmalloc/echo-server:latest

docker run -d --name web3 --net mynetwork jmalloc/echo-server:latest

```

## HAProxy config file

```cfg
global
  stats socket /var/run/api.sock user haproxy group haproxy mode 660 level admin expose-fd listeners
  log stdout format raw local0 info

defaults
  mode http
  option  httplog
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s
  log global

frontend stats
  bind *:8404
  stats enable
  stats uri /
  stats refresh 10s

frontend prometheus
  bind *:8405
  mode http
  http-request use-service prometheus-exporter if { path /metrics }

frontend myfrontend
  bind :80
  default_backend webservers

backend webservers
  server s1 web1:8080 check
  server s2 web2:8080 check
  server s3 web3:8080 check

```

Enable stats, metrics and do the usual haproxy work, load balancing towards the servers

## Running HAProxy

NB: Haproxy official images can be found at Docker and HAProxy Tech. I'll go with the Haxproxy tech one.

```bash
docker run -d \
   --name haproxy \
   --net mynetwork \
   -p 8080:80 \
   -p 8404:8404 \
   -p 8405:8405 \
   haproxytech/haproxy-alpine
```

## Test

open http://localhost:8080

## Stats

open http://localhost:8404

## Metrics

open http://localhost:8405

## Shutdown the test env

```bash
docker stop web1 && docker rm web1
docker stop web2 && docker rm web2
docker stop web3 && docker rm web3
docker stop haproxy && docker rm haproxy
docker network rm mynetwork
```
