Hippo CMS Docker Deployments
=============================
Deploy Hippo CMS as containers using Docker Compose, Docker Swarm and Kubernetes. I have been working with Hippo CMS platform since early 2015. Hippo CMS is by far one of the finest Java-based Open Source Content Management system out there. Ever since I have learned about Docker, I have been experimenting with the idea of deploying Hippo CMS on Docker. After stalling and failing for years, I finally have the recipe for successfully deploying Hippo CMS as a Dockerized Container using Docker Compose, Docker Swarm and Kubernetes (using OpenShift and Rancher). 

* [Docker Compose](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/docker-compose/README.md)
* [Docker Swarm](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/docker-swarm)
* [Kubernetes](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/kubernetes)
  * [OpenShift](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/openshift/README.md)

TBD (Stay Tuned ... currently working on it)
 * [Rancher]

The key objective here is to seperate the scalable deployment concernes from the core application itself, therefore, changes to the application platform should not influence the deployment model as long as [Hippo CMS architecture](https://www.onehippo.org/library/architecture/hippo-cms-architecture.html) remains the same.
```
                                                    
       +-------+                +-------+         
       |Browser|                |Browser|             
       +-------+                +-------+          
           |                         |               
           |                         |                
  +--------------------------------------------+     
  |        |                         |         |
  |  +------------+            +-----------+   |      
  |  |    CMS     |            |   Site    |   |      
  |  +------------+            +-----------+   |      
  |        |                         |         |       
  |        +-------------------------+         |       
  |                     |                      |      
  |                     |                      |       
  |          +--------------------+            |      
  |          |   JCR Repository   |            |     
  |          +--------------------+            |      
  |                     |                      |      
  |                     |                      |    
  |          +--------------------+            |    
  |          |        RDBMS       |            |     
  |          +--------------------+            |   
  +--------------------------------------------+    
```

Desclaimer
==========
* All ideas and concepts presented here are my own and may NOT have been tested, approved and supported by the parent company of Hippo CMS.

