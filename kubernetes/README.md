# Hippo CMS -- Deployment using Kubernetes
*Please note: Documentation is still being added to this page, I appreciate your patience.*
When it comes to Kubernetes, we talk about pods, services etc. From principles of Kubernetes, Pod is the minium unit of deployment, a pod can contain more than one container that can share same volume and can communicate with other containers on localhost. Based on what we know about Apache Jackrabbit cluster node specs (Hippo CMS uses Apache Jackrabbit repository), nodes in a cluster should have their own repository for maintaining the local repository index, therefore, a pod can contain only one Hippo CMS conatiner. 


Architecture
----------
```
                    |                                                                    
                    |                           +---------------+                        
  Database Service  |                           |     MySQL     |                        
                    |                           +-------|-------+                        
                    |                                   |                                
 ------------------------          +--------------------|----------------------+         
                    |              |                    |                      |         
                    |              |                    |                      |         
                    |     +----------------+    +----------------+     +----------------+
                    |     | Hippo Pod      |    | Hippo Pod      |     | Hippo Pod      |
                    |     |+--------------+|    |+--------------+|     |+--------------+|
  Hippo CMS Service |     ||   Hippo CMS  ||    ||   Hippo CMS  ||     ||   Hippo CMS  ||
                    |     |+--------------+|    |+--------------+|     |+--------------+|
                    |     |+--------------+|    |+--------------+|     |+--------------+|
                    |     ||   Nginx      ||    ||   Nginx      ||     ||   Nginx      ||
                    |     |+--------------+|    |+--------------+|     |+--------------+|
                    |     +----------------+    +-------|--------+     +--------|-------+
                    |             |                     |                       |        
                    |             +---------------------------------------------+        
------------------------                                |                                
                    |                                   |                                
         Ingress    |                        +-----------------------+                   
                    |                        | Kubernetes Ingress LB |                   
                    |                        +-----------------------+                   
                    |                                                                    
                    |                                                                    
                                          
```

To deploy
---------
First create a namespace, lets call it **hippo-test**
```
kubectl create -ns hippo-test
```

Next, deploy the application to namespace **hippo-test**
```
kubectl create -f hippo-msql.yaml -n hippo-test
```
In order to access the service, you will have to install Ingress Controller and then create an ingress proxy service. 

References
-------
* Get started with Kubernetes: https://kubernetes.io/docs/tasks/tools/install-kubectl/
* Kubernetes Ingress Controller: https://kubernetes.github.io/ingress-nginx/deploy/
