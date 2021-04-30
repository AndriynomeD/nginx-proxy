
nginx-proxy sets up a container running nginx and [docker-gen][1].  docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped.

See [Automated Nginx Reverse Proxy for Docker][2] for why you might want to use this.

### Origin Repository

This repo is fork of [jwilder/nginx-proxy][origin-repo] so you need read origin md file

### What included

Include next containers:
1. jwilder/nginx-proxy
2. docker.elastic.co/elasticsearch/elasticsearch:7.7.1 - for shared access from multiple magento instances (need use different "Elasticsearch Index Prefix" in each instance)
~~3. docker.elastic.co/kibana/kibana:7.7.1 - data visualization plugin for Elasticsearch~~ (Commented/Disabled)
4. phpmyadmin/phpmyadmin - manage databases

### Usage

1. Install docker & docker-compose
2. Add user into group docker & www-data
    1. Run commands:
    ```shell
        $ sudo usermod -aG docker ${USER}
        $ sudo usermod -aG www-data ${USER}
    ```
    2. Restart system
3. Create docker external network (nginx-proxy - grouped container for reverse nginx, elastic-net - for share elasticsearch, databases - grouped db for phpmyadmin):
    ```shell
        $ docker network create nginx-proxy
        $ docker network create elastic-net
        $ docker network create databases
    ```
4. Add share docker service to `/etc/hosts` file:
    ```
    #share docker service
    127.0.0.1 elasticsearch eskibana phpmyadmin
    ```
5. Choose shared elasticsearch version & update docker-compose.yml (Default version is 7.7.1).
   In docker-compose.yml for elasticsearch available `<elasticsearch-verion>`:
    - elasticsearch5 - 5.6
    - elasticsearch6 - 6.8.15
    - elasticsearch7 - 7.7.1
   ```dockerfile
   elasticsearch:
    container_name: elasticsearch
    hostname: elasticsearch.docker
    build:
        context: <elasticsearch-verion>/   (example `context: elasticsearch7/`)
    ......
    esdata:
        image: tianon/true
        volumes:
            - ./persistent/<elasticsearch-verion>:/usr/share/elasticsearch/data
    ```

6. Update java param for elasticsearch
    ```shell
    $ sudo gedit /etc/sysctl.conf
    ```
    Add 'vm.max_map_count=262144' into file & save it.

7. Up docker containers:
    ```shell
        $ docker-compose up -d
    ```

### Else
Then start any containers you want proxied with an env var `VIRTUAL_HOST=subdomain.youdomain.com`

In docker-compose.yml file edit phpMyAdmin config if need add additional db:
1. Check full name of docker container you want to add

    $ docker network ls

2. Add container full name to phpMyAdmin config (example we already have instance_db_1 and add instance_db_2 to config):
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

### Access to Elasticsearch, Kibana, phpMyAdmin

http://elasticsearch:9200  
http://eskibana:5601  
http://phpmyadmin:8080  

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80. 
If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.
If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

### Maybe useful

1. Delete container not used longer that a week:
```
    $ docker ps --filter "status=exited" | grep -E "Exited .*week[s]? ago" | awk '$2 != "tianon/true" {print $1}' | xargs --no-run-if-empty docker rm
```

### Problem

1. Some time docker return 'random' ip then varnish ask for 'web' container ip - because all varnish & web container grouped in nginx-proxy network docker not clear know what exactly 'web' container want specific 'varnish' container.

  [1]: https://github.com/jwilder/docker-gen
  [2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/
  [origin-repo]: https://github.com/jwilder/nginx-proxy