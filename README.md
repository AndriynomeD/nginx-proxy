# nginx-proxy & shared infrastructure for [Magento 2 Docker][docker-magento2]

Magento 2 Docker infrastructure have the following structure:
```
nginx-proxy & shared infrastructure
    ├── project1 infrastructure
    ├── project2 infrastructure
    ├── ...
    ├── projectN infrastructure
    └── ...
```

### What included

1. [nginx-proxy/nginx-proxy][nginx-proxy-repo] sets up a container running nginx and [docker-gen][1]. docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped. See [Automated Nginx Reverse Proxy for Docker][2] for why you might want to use this.
2. elasticsearch - for shared access from multiple magento instances (need use different "Elasticsearch Index Prefix" in each instance)
3. kibana for elasticsearch (Commented/Disabled by default)
3. opensearch - for shared access from multiple magento instances (need use different "Opensearch Index Prefix" in each instance)
3. opensearch-dashboards for elasticsearch (Commented/Disabled by default)
4. phpMyAdmin - databases manager


### Usage

1. Install docker & docker-compose
2. Add user into group docker & www-data
    1. Run commands:
    ```shell
   sudo usermod -aG docker ${USER}
   sudo usermod -aG www-data ${USER}
    ```
    2. Restart system
3. Create docker external network (nginx-proxy - grouped container for reverse nginx, elastic-net - for share elasticsearch, databases - grouped db for phpmyadmin):
    ```shell
   docker network create nginx-proxy
   docker network create elastic-net
   docker network create search-engine-net
   docker network create databases
    ```
4. Add share docker service to `/etc/hosts` file:
    ```hosts
    #share docker service
    127.0.0.1 elasticsearch eskibana opensearch opensearch-dashboards phpmyadmin
    ```
5. Copy `docker-compose.yml.sample` as `docker-compose.yml`. Optional:
   - Uncoment eskibana, opensearch-dashboards if needed. Comment elasticsearch/opensearch if someone not needed
   - Choose shared elasticsearch/opensearch version & update docker-compose.yml. Or update with you're version.
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
     volumes:
       - ./persistent/<elasticsearch-verion>:/usr/share/elasticsearch/data
   ```
6. Update java param for elasticsearch/opensearch
    ```shell
    sudo gedit /etc/sysctl.conf
    ```
    Add `vm.max_map_count=262144` into file & save it.
7. Up docker containers:
    ```shell
    docker-compose up -d
    ```
   P.S. If something went wrong try 1-7 step & force recreate containers:
   ```shell
   docker-compose up --build --force-recreate
   docker-compose up -d
    ```

### Add to phpMyAdmin dropdown additional db

1. Check project db container hostname you want to add in project docker-compose.yml
2. Add db container hostname to phpMyAdmin config (example we already have hostname_db1 and add hostname_db2 to config):
```dockerfile
phpmyadmin:
  ...
  environment:
    PMA_HOSTS: hostname_db1,hostname_db2
  ...
```

### Access to Elasticsearch, Kibana, phpMyAdmin

http://phpmyadmin:8080  
http://elasticsearch:9200  
http://eskibana:5601  
http://opensearch:19200  
http://opensearch-dashboards:15601

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80. 
If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.
If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

[1]: https://github.com/nginx-proxy/docker-gen
[2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/
[nginx-proxy-repo]: https://github.com/nginx-proxy/nginx-proxy
[nginx-proxy]: https://github.com/AndriynomeD/nginx-proxy
[docker-magento2]: https://github.com/AndriynomeD/docker-magento2