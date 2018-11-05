Hippo CMS Docker Deployments
=============================
I have been working with Hippo CMS platform since early 2015. Hippo CMS is by far one of the finest Java-based Open Source Content Management system out there. Ever since I have learned about Docker, I have been experimenting with the idea of deploying Hippo CMS on Docker:
* Simplify the overall deployment process for one or more reasons listed below, 
* CI/CD automation,
  * Use [Jenkins](https://wiki.jenkins.io/display/JENKINS/Docker+Plugin) to build Docker Images
  * Use [Sonatype Nexus Repository](https://help.sonatype.com/repomanager3/private-registry-for-docker) as a private registry to store Docker Images. 
* Run scalable application cluster with Horizontally scalable infrastructure.

After stalling and failing for years, I finally have the recipe for successfully deploying Hippo CMS as a Dockerized Container using, 
* Docker Compose
* Docker Swarm and 
* Kubernetes 
 * OpenShift 
 * Rancher 

While Docker Compose is great for running test and demo clusters, Docker Swarm and Kubernetes could be a great fit for a production cluster. Using Openshift and Rancher Orchestration platforms -- you can even run multiple clusters (multi-tenant) of the same application and manage them more elegantly and effortlessly. 

Using these deployment schemes, you can achieve run highly scalable cluster of web applications. Using Kubernetes, you can achieve even dynamic auto-scaling capabilities. 

The links below will take you to the documentation of specific deployment schemes, choose one or more that suits you the best.

* [Docker Compose](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/docker-compose/README.md)
* [Docker Swarm](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/docker-swarm)
* [Kubernetes](https://github.com/maheshacharya/hippo-docker-deployments/tree/master/kubernetes)
  * [OpenShift](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/openshift/README.md)
  * [Rancher](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/rancher/README.md)

The key objective here is to separate the scalable deployment concerns from the core application itself, therefore, changes to the application platform should not influence the deployment model as long as [Hippo CMS architecture](https://www.onehippo.org/library/architecture/hippo-cms-architecture.html) remains the same.
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
Open Source -- Free Software
-----------------
All of the tools and technologies used here are Free and Open Source (some do offer enterpise licensing for supprort and advanced features). All you need is a problem that requires large scale deployement solution -- and of course a lot of time and energy to research and ask a lot of stupid questions on community forums -- ah, well, there is no such thing called *stupid question*. 


Related References:
----------
* [Build Docker Images of your Hippo CMS project](https://medium.com/@maheshacharya_44641/hippo-cms-docker-containerization-703e2e4e496c).
* [Install OpenShift Origin on Ubuntu 18.04](https://medium.com/@maheshacharya_44641/install-openshift-origin-on-ubuntu-18-04-7b98773c2ee6).
* [Rancher 2.0 Documentation](https://rancher.com/docs/rancher/v2.x/en/)
* [Docker Swarm](https://docs.docker.com/engine/swarm/).
* [Hippo CMS Documentation](https://www.onehippo.org/library/about/introduction-hippo.html)
* [Kubernetes Documentation](https://kubernetes.io/docs/home/?path=browse)
* [Docker Getting Started](https://docs.docker.com/get-started/)
* [Traefik](https://docs.traefik.io/)
* [Apache Jackrabbit](http://jackrabbit.apache.org/jcr/index.html)
* [Sonatype Nexus - Private Registry for Docker](https://help.sonatype.com/repomanager3/private-registry-for-docker)
* [Jenkins Docker Plugin](https://wiki.jenkins.io/display/JENKINS/Docker+Plugin)


Disclaimer
-------
* All ideas and concepts presented here are my own and may NOT have been tested, approved and supported by the parent company of Hippo CMS.

