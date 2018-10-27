Deploy Hippo CMS using Docker Compose
=====================================
Read more on Docker Compose documentation here: https://docs.docker.com/compose/

Deployment Architecture
-----------------------
```
                                                  +-----------+
                      |                           |   MySQL   |
    Database          |                           +-----^-----+
                      |                                 |
   ----------- -------|                                 |
                      |            +-------------------++-------------------+
                      |            |                    |                   |
    Hippo CMS         |    +-------+------+       +-----+------+    +-------+------+
                      |    |   Node #1    |       |   Node #2  |    |   Node #3    |
                      |    +-------^------+       +-----+------+    +-------^------+
   -------------------|            |                    |                   |
                      |            +--------------------+-------------------+
                      |                                 |
    LB/Reverse Proxy  |                           +-----+------+
                      |                           |  Apache    |
                      |                           +------------+
                      
```
This deployment architecture contains:
* MySQL running in a single container with host volume mapping for persistent storage.
* Three instances of Hippo CMS running in separate containers with links to MySQL container
* Apache (httpd) running as a reverse proxy and load balancer. 

Container Images
----------------
* MySQL : https://hub.docker.com/_/mysql/
* Hippo Project: https://hub.docker.com/r/maheshacharya/myhippoproject-docker-deployment-demo-mysql/
* Apache2 : https://hub.docker.com/r/rgoyard/apache-proxy/



Apply Configuration Changes
---------------------------
Modify *[default](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/docker-compose/apache2/sites-available/default)* and *[default-ssl](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/docker-compose/apache2/sites-available/default-ssl)* file under *[apache2/sites-available](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/docker-compose/apache2/sites-available).* to add proper server/domain name:
Replace ```<domain name>``` with server DNS name, for example: example.com, cms.example.com etc.


Deploy Hippo CMS
----------
Change current working directory to docker-compose and then run:
```
$ docker-compose up -d
```
