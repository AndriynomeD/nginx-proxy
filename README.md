![latest 0.7.0](https://img.shields.io/badge/latest-0.7.0-green.svg?style=flat)
![nginx 1.17.8](https://img.shields.io/badge/nginx-1.17.8-brightgreen.svg) ![License MIT](https://img.shields.io/badge/license-MIT-blue.svg) [![Build Status](https://travis-ci.org/jwilder/nginx-proxy.svg?branch=master)](https://travis-ci.org/jwilder/nginx-proxy) [![](https://img.shields.io/docker/stars/jwilder/nginx-proxy.svg)](https://hub.docker.com/r/jwilder/nginx-proxy 'DockerHub') [![](https://img.shields.io/docker/pulls/jwilder/nginx-proxy.svg)](https://hub.docker.com/r/jwilder/nginx-proxy 'DockerHub')


nginx-proxy sets up a container running nginx and [docker-gen][1].  docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped.

See [Automated Nginx Reverse Proxy for Docker][2] for why you might want to use this.

### Origin Repository

This repo is fork of [jwilder/nginx-proxy][origin-repo] so you need read origin md file

### What included

Include next containers:
1. jwilder/nginx-proxy
2. docker.elastic.co/elasticsearch/elasticsearch:5.2.1 - for shared access from multiple magento instances (need use different "Elasticsearch Index Prefix" in each instance)
3. docker.elastic.co/kibana/kibana:5.2.1 - data visualization plugin for Elasticsearch

### Some docker command info

For use docker/docker-compose without sudo (need relogin into system):

    $ sudo usermod -aG docker {{user}}

Delete docker network:

    $ docker network rm {{network_name}}

Show list of docker networks:

    $ docker network ls

### Usage

Create docker external network:

    $ docker network create nginx-proxy

Add next to `/etc/hosts` file:
```
# share docker service
127.0.0.1 elasticsearch
127.0.0.1 eskibana
```

Make directory `persistent/elasticsearch` inside root directory

To run it:

    $ docker-compose up
    
If get kibana error try on host machine:

    $ sudo sysctl -w vm.max_map_count=262144

Then start any containers you want proxied with an env var `VIRTUAL_HOST=subdomain.youdomain.com`

For use in Magento Elasticsearch engine change next config values:
1. Search Engine: Elasticsearch 5.0+
2. Elasticsearch Server Hostnam: elasticsearch
3. Elasticsearch Server Port: 9200
4. Elasticsearch Index Prefix - must be unique

### Access to Elasticsearch & Kibana

http://elasticsearch:9200
http://eskibana:5601

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80.  If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.  If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

  [1]: https://github.com/jwilder/docker-gen
  [2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/
  [origin-repo]: https://github.com/jwilder/nginx-proxy