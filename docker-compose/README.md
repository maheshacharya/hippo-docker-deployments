Deploy Hippo CMS using Docker Compose
=====================================
Hippo CMS is one of the simple yet sophisticated Open Source Java web application platform, procedures to properly deploy Hipo CMS application requires covering a lot of areas of Dev-Ops tooling and concepts. Hippo CMS uses Apache Jackrabbit JCR repository, achieving a scalable deployment procedure makes it a true enterprice ready content management platform.


Goals
=====
* Deploy containerized Hippo CMS application on a single node Docker host VM using Docker Compose.
* Use MySQL Database as persistance data storage for the repository.
* Run 3 instances of Hippo CMS for to achieve a desired scalability of website and cms applications. 
* Run Apache as Load Balancer and Reverse Proxy.
  * Use round-robin load balancing with stickness enabled.
  * Stickiness is important for CMS application as it is stateful. 


Requirements
============
* Linux Server VM (Ubuntu, RHEL, CentOS etc.)
* Docker
* Docker Compose

Our test platform specs
=======================
* Host VM
 * Ubuntu 18.04 on AWS 
* Docker version: 18.06.1-ce
* Docker Compose Version: 1.22.0  


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
Replace ```cloud-hub.co``` with your own server DNS name.


Deploy Hippo CMS
----------------
Change current working directory to docker-compose and then run:
```
$ docker-compose up -d
```
Terminal Output 
---------------
Successful deployment out will resemble something like below:

```
Creating network "docker-compose_default" with the default driver
Creating docker-compose_hippo-mysql-database_1 ... done
Creating docker-compose_hippo3_1               ... done
Creating docker-compose_hippo2_1               ... done
Creating docker-compose_hippo1_1               ... done
Creating docker-compose_apache_1               ... done
```


Remove Appication Stack
-----------------------
```
$ docker-compose down
```
Terminal Output
---------------
Successful removal will display following output in terminal

```
Stopping docker-compose_apache_1               ... done
Stopping docker-compose_hippo3_1               ... done
Stopping docker-compose_hippo1_1               ... done
Stopping docker-compose_hippo2_1               ... done
Stopping docker-compose_hippo-mysql-database_1 ... done
Removing docker-compose_apache_1               ... done
Removing docker-compose_hippo3_1               ... done
Removing docker-compose_hippo1_1               ... done
Removing docker-compose_hippo2_1               ... done
Removing docker-compose_hippo-mysql-database_1 ... done
Removing network docker-compose_default
```

Reference Links
===============
* Docker: https://docs.docker.com/
* Docker Compose: https://docs.docker.com/compose/
* Hippo CMS: https://www.onehippo.org/trails/getting-started/hippo-essentials-getting-started.html



