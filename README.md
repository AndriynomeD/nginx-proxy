
nginx-proxy sets up a container running nginx and [docker-gen][1].  docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped.

See [Automated Nginx Reverse Proxy for Docker][2] for why you might want to use this.

### Origin Repository

This repo is fork of [jwilder/nginx-proxy][origin-repo] so you need read origin md file

### What included

Include next containers:
1. jwilder/nginx-proxy
2. docker.elastic.co/elasticsearch/elasticsearch:5.2.1 - for shared access from multiple magento instances (need use different "Elasticsearch Index Prefix" in each instance)
3. docker.elastic.co/kibana/kibana:5.2.1 - data visualization plugin for Elasticsearch
4. phpmyadmin/phpmyadmin - manage databases

### Some docker command info

For use docker/docker-compose without sudo (need relogin into system):

    $ sudo usermod -aG docker {{user}}

Delete docker network:

    $ docker network rm {{network_name}}

Show list of docker networks:

    $ docker network ls

### Usage

Create docker external network (nginx-proxy - grouped container for reverse nginx, elastic-net - for share elasticsearch, databases - grouped db for phpmyadmin):

    $ docker network create nginx-proxy
    $ docker network create elastic-net
    $ docker network create databases

Add next to `/etc/hosts` file:
```
# share docker service
127.0.0.1 elasticsearch
127.0.0.1 eskibana
127.0.0.1 phpmyadmin
```

Make directory `persistent/elasticsearch` inside root directory

To run it:

    $ docker-compose up
    
If get kibana error try on host machine:

    $ sudo sysctl -w vm.max_map_count=262144

Then start any containers you want proxied with an env var `VIRTUAL_HOST=subdomain.youdomain.com`

In docker-compose.yml file edit phpMyAdmin config if need add additional db:
1. Check full name of docker container you want to add

    $ docker network ls

2. Add container full name to phpMyAdmin config (example we already have instance_db_1 and add instance_db_2 to config):
Before:
```
 phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - 8080:80
    environment:
      PMA_HOSTS: instance_db_1
      PMA_PORTS: 3306
    external_links:
      - instance_db_1
    networks:
      - databases
```
After:
```
 phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - 8080:80
    environment:
      PMA_HOSTS: instance_db_1,instance_db_2
      PMA_PORTS: 3306,3306
    external_links:
      - instance_db_1
      - instance_db_2
    networks:
      - databases
```

For use in Magento Elasticsearch engine change next config values:
1. Search Engine: Elasticsearch 5.0+
2. Elasticsearch Server Hostnam: elasticsearch
3. Elasticsearch Server Port: 9200
4. Elasticsearch Index Prefix - must be unique

### Access to Elasticsearch, Kibana, phpMyAdmin

http://elasticsearch:9200
http://eskibana:5601
http://phpmyadmin:8080

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80.  If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.  If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

### Minimized config

File docker-compose-min.yml contain config without varnish & cron (you als can commented unneaded)

### Problem

Some time docker return 'random' ip then varnish ask for 'web' container ip - because all varnish & web container grouped in nginx-proxy network docker not clear know what exactly 'web' container want specific 'varnish' container.

  [1]: https://github.com/jwilder/docker-gen
  [2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/
  [origin-repo]: https://github.com/jwilder/nginx-proxy