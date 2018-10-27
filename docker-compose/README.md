Deploy Hippo CMS using Docker Compose
=====================================
[Hippo CMS](https://www.onehippo.org/) is one of the elegant yet sophisticated Open Source Java web application platform, procedures to properly deploy Hippo CMS application requires covering a lot of areas of Dev-Ops tooling and concepts. Hippo CMS uses Apache Jackrabbit JCR repository, achieving a scalable deployment procedure makes it a true enterprise-ready content management platform.


Goals
-----
* Deploy containerized Hippo CMS application on a single node Docker host VM using Docker Compose.
* Use MySQL Database as persistent data storage for the repository.
  * Use volume mounted on the Host VM so that re-deployment of application will bootstrap from the data on the persistent volume. 
* Run 3 instances of Hippo CMS to achieve the desired scalability of website and cms applications. 
* Run Apache (httpd) as Load Balancer and Reverse Proxy.
  * Use round-robin load balancing with sickiness enabled.
  * Stickiness is important for CMS application as it is stateful. 


Requirements
------------
* Linux Server VM (Ubuntu, RHEL, CentOS etc.)
* Docker
* Docker Compose

Our test platform specs
-----------------------
* Host VM
 * Ubuntu 18.04 on AWS 
* Docker version: 18.06.1-ce
* Docker Compose Version: 1.22.0  
* Hippo CMS version: 12.04

Hippo CMS Poject
----------------
We are using this project https://github.com/maheshacharya/hippo-docker-example
This project has two maven profiles that enable building Docker Images. To use your own projects, you will have to build Docker Image of your projects and then deploy the images to [Docker Hub](https://hub.docker.com/) or to a private registry.
We have already created an image that is needed for this deployment which is available [here](https://hub.docker.com/r/maheshacharya/myhippoproject-docker-deployment-demo-mysql/)


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
---------------
* Docker: https://docs.docker.com/
* Docker Compose: https://docs.docker.com/compose/
* Hippo CMS: https://www.onehippo.org/trails/getting-started/hippo-essentials-getting-started.html
* Hippo CMS Demo Project with Container Image building profile: https://github.com/maheshacharya/hippo-docker-example

Limitations of Docker Compose based deployment
----------------------------------------------
Although Docker Compose based deployment simplifies the overall deployment process, the Application cannot be dynamically scaled: Hippo CMS containers cannot be scaled at runtime, adding more containers will require changes to the Load Balancer configuration and the containers have to be re-deployed to bind any configuration changes. To achieve dynamic scalability it is better to use Docker Swarm or Kubernetes as your deployment platform. 

Conclusion
----------
Docker Compose based deployment model simplifies the overall repeatable deployment process with the execution of a single command.


