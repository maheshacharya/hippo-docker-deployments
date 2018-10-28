# Hippo CMS -- Deployment using Docker Swarm
Docker Swarm is an elegant platform for deploying a highly-available  and scalable cluster of Hippo CMS application. You can run 100's of containers of Hippo CMS instances at runtime, scale-up or scale-down based on the processing requirements. You can scale underlying infrastructure horizontally by adding new resoures (VM nodes). 

Goals
-----
* Use MySQL persistence storage for the repository.
* Start the cluster with 4 containers of Hippo CMS.
* Use [Traefik](https://traefik.io/) load balancer to acheive JSESSIONID based session affinity.
* Use Apache(http) reverse proxy.

Our test platform specs
---

* Host VM -- Ubuntu 18.04 on AWS -- Single node VM (Master). 
* Docker version: 18.06.1-ce
* Hippo CMS version: 12.04


Architecture
------------
```
                                                  +----------+
                                                  |   MySQL  |
    Database          |                           +----------+
                      |                                 |
   ----------- -------                                  |
                      |            +-------------------++-------------------+
                      |            |                    |                   |
    Hippo CMS         |    +-------+------+       +-----+------+    +-------+------+
                      |    |   Node #1    |       |   Node #2  |    |   Node #3    |
                      |    +-------^------+       +-----+------+    +-------^------+
   -------------------             |                    |                   |
                      |            +--------------------+-------------------+
                      |                                 |
                      |                           +-----+------+
    Load Balancer     |                           | Traefik LB |
                      |                           +-----^------+
   -------------------                                  |
                      |                                 |
                      |                        +--------+---------+
    Reverse Proxy     |                        | Apache (httpd)   |
                                               +------------------+
```
Container Images
------------------

* MySQL : https://hub.docker.com/_/mysql/
* Hippo Project: https://hub.docker.com/r/maheshacharya/myhippoproject-docker-deployment-demo-mysql/
* Apache2 : https://hub.docker.com/r/rgoyard/apache-proxy/
* Traefik: https://hub.docker.com/_/traefik/

Apply Configuration Changes
---------------------------
Modify default file under **apache2/sites-available**. to add proper server/domain name: Replace **cloud-hub.co** with your own server DNS name.


Deploy the stack
-----------------
```
cd docker-swarm
docker swarm init
docker stack deploy -c docker-compose.yml hippo
```
