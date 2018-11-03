# Hippo CMS -- Deployment using Docker Swarm
Docker Swarm is an elegant platform for deploying a highly-available  and scalable cluster of Hippo CMS application. You can run 100's of containers of Hippo CMS instances at runtime, scale-up or scale-down based on the processing requirements. You can scale underlying infrastructure horizontally by adding new resoures (VM nodes). 

Goals
-----
* Use MySQL persistence storage for the repository.
* Start the cluster with 4 containers of Hippo CMS.
* Use [Traefik](https://traefik.io/) load balancer to achieve JSESSIONID based session affinity.
* Use Apache(http) reverse proxy.

Our test platform specs
---

* Host VM -- Ubuntu 18.04 on AWS -- Single node Swarm Cluster(Master). 
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

Preparation
-----------
* Copy "apache2" folder(incluing all subfolder/files) to $HOME directory. 
    * You can also move to another location, but you will have to change the corresponding path in yhe yaml file. 
* Make "hippo/mysql" directory under $HOME directory, without this MySQL service, will fail. 


Deploy the stack
-----------------
```
cd docker-swarm
docker swarm init
docker stack deploy -c docker-compose.yml hippo
```

Successful deployment will show this output in terminal.
--------

```
Creating network hippo_net
Creating network hippo_mysql
Creating network hippo_traefik
Creating service hippo_apache
Creating service hippo_hippo
Creating service hippo_loadbalancer
Creating service hippo_mysql-database
```
Validated that services are running
---------
Running ```docker service ls``` will show similar outout(below).

```
ID                  NAME                   MODE                REPLICAS            IMAGE                                                              PORTS
u63qa9leaegc        hippo_apache           replicated          1/1                 rgoyard/apache-proxy:latest                                        *:80->80/tcp, *:443->443/tcp
h9136pezctbk        hippo_hippo            replicated          4/4                 maheshacharya/myhippoproject-docker-deployment-demo-mysql:latest   *:30040->8009/tcp, *:30041->8080/tcp, *:30042->9443/tcp
c5in1wo7z62f        hippo_loadbalancer     replicated          1/1                 traefik:latest                                                     *:9090->8080/tcp, *:30043->80/tcp
h5r6kunnq1py        hippo_mysql-database   replicated          1/1                 mysql:5.6.36                                                       *:30044->3306/tcp

```
Scaling Hippo CMS  to 8 containers.
-------

Make sure the underlying cluster VM has enough resources to support the new instances, othewrise, you can join more VM nodes to swram cluster to scale out and then scale the containers.
```
docker service scale hippo_hippo=8
```
**Output**

```
hippo_hippo scaled to 8
overall progress: 8 out of 8 tasks 
1/8: running   [==================================================>] 
2/8: running   [==================================================>] 
3/8: running   [==================================================>] 
4/8: running   [==================================================>] 
5/8: running   [==================================================>] 
6/8: running   [==================================================>] 
7/8: running   [==================================================>] 
8/8: running   [==================================================>] 
verify: Service converged 

```
Removing the Stack
-----------------
```
docker stack rm hippo

```
**Output**
```
Removing service hippo_apache
Removing service hippo_hippo
Removing service hippo_loadbalancer
Removing service hippo_mysql-database
Removing network hippo_mysql
Removing network hippo_net
Removing network hippo_traefik
```
References
----------
* Docker Swarm: https://docs.docker.com/engine/swarm/
