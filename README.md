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
2. [aegypius/mkcert-for-nginx-proxy][mkcert-repo] is a lightweight companion container for the [nginx-proxy/nginx-proxy][nginx-proxy-repo]. It's heavily inspired by [nginx-proxy/acme-companion][3] and it allows the creation/renewal of self-signed certificate with a root certificate authority.
3. phpMyAdmin - databases manager
4. Mailpit as shared service for email testing for developers.
5. ElasticSearch - for shared access from multiple magento instances (need use different "Elasticsearch Index Prefix" in each instance)
6. OpenSearch - for shared access from multiple magento instances (need use different "Opensearch Index Prefix" in each instance)
7. Kibana for ElasticSearch. Disabled by default
8. OpenSearch Dashboards for OpenSearch. Disabled by default

P.S. Shared Services phpMyAdmin, Mailpit, ElasticSearch, OpenSearch, Kibana, OpenSearch can be enabled/disabled by comment/uncomment  `profiles: ["disabled"]`

### Usage

1. Install Docker & Docker Compose
2. Add user into group docker & www-data
    1. Run commands:
    ```shell
   sudo usermod -aG docker ${USER}
   sudo usermod -aG www-data ${USER}
    ```
    2. Restart system
3. Create docker external network:
    ```shell
   docker network create nginx-proxy
   docker network create search-engine-net
   docker network create databases
   docker network create mail-services
    ```
   Explanation: 
   - nginx-proxy - grouped container for reverse nginx;
   - search-engine-net - provide access to shared elasticsearch/opensearch;
   - databases - grouped db for phpmyadmin;
   - mail-services - provide access to Mailpit

4. Add share docker service to `/etc/hosts` file:
    ```hosts
    #share docker service
    127.0.0.1 phpmyadmin.shared webmail.shared elasticsearch.shared opensearch.shared eskibana.shared opensearch-dash.shared
    ```
5. Copy `docker-compose.yml.sample` as `docker-compose.yml`. Optional:
   - Enabled/Disabled shared services (phpMyAdmin, Mailpit, ElasticSearch, OpenSearch, Kibana, OpenSearch) by comment/uncomment  `profiles: ["disabled"]`
   - Choose shared elasticsearch/opensearch version & update docker-compose.yml. Or update Dockerfile with preferred version. In `docker-compose.yml` for elasticsearch available `<elasticsearch-verion>`:
     - elasticsearch5 - 5.6
     - elasticsearch6 - 6.8.15
     - elasticsearch7 - 7.7.1
   ```dockerfile
   elasticsearch:
     ...
     build:
       context: containers/<elasticsearch-verion>/   (example `context: containers/elasticsearch7/`)
     volumes:
       - ./volumes/<elasticsearch-verion>:/usr/share/elasticsearch/data
   ```
6. Update java param for elasticsearch/opensearch
    ```shell
    sudo gedit /etc/sysctl.conf
    ```
    Add `vm.max_map_count=262144` into file & save it.
7. Up docker containers:
    ```shell
    docker compose up -d
    ```
   mkcert service will automatically create CA root certificate & certificates for shared services
   P.S. If something went wrong try 1-7 step & force recreate containers:
   ```shell
   docker compose up --build --force-recreate
   docker compose up -d
    ```
8. Import root CA certificate into browsers to remove warns about invalid/untrusted SSL certificate or project domain that use https:
   - For Chrome on Linux: go to Chrome Settings -> Privacy And Security -> Security -> Manage Certificates -> Custom: Installed by you -> Trusted Certificates press `Import` btn and select `{{nginx-proxy_folder}}/etc/mkcert/root_certs/rootCA.pem` (`{{nginx-proxy_folder}}` - path to this repo folder)

### Access to phpMyAdmin Mailtip, ElasticSearch, OpenSearch, Kibana, OpenSearch Dashboard

- phpMyAdmin (https://phpmyadmin.shared/)
- Mailtip (https://webmail.shared/)
- ElasticSearch (https://elasticsearch.shared/)
- Kibana (https://eskibana.shared/)
- OpenSearch (https://opensearch.shared/)
- OpenSearch Dashboard (https://opensearch-dash.shared/)

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

### Multiple Ports

If your container exposes multiple ports, nginx-proxy will default to the service running on port 80. 
If you need to specify a different port, you can set a VIRTUAL_PORT env var to select a different one.
If your container only exposes one port and it has a VIRTUAL_HOST env var set, that port will be selected.

[1]: https://github.com/nginx-proxy/docker-gen
[2]: http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/
[3]: https://github.com/nginx-proxy/acme-companion
[nginx-proxy-repo]: https://github.com/nginx-proxy/nginx-proxy
[mkcert-repo]: https://github.com/aegypius/mkcert-for-nginx-proxy
[nginx-proxy]: https://github.com/AndriynomeD/nginx-proxy
[docker-magento2]: https://github.com/AndriynomeD/docker-magento2