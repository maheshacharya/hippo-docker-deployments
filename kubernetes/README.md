# Hippo CMS -- Deployment using Kubernetes
*Please note: Documentation is still being added to this page, I appreciate your patience.*

When it comes to Kubernetes, we talk about pods, services etc. From principles of [Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), a **Pod** is a minimum unit of deployment: a **Pod** can contain more than one container that can share same volume and can communicate with other containers on localhost. 

Based on what we know about [Apache Jackrabbit](https://wiki.apache.org/jackrabbit/Clustering) cluster node specs (Hippo CMS uses Apache Jackrabbit repository), nodes in a cluster should have their own repository for maintaining the local repository index, therefore, a pod can contain only one Hippo CMS conatiner. 


Architecture
----------
```
                                                                                                        
                    |                                                                                   
                    |                                   +-------------+                                 
   Database Service |                                   |    MySQL    |                                 
                    |                                   +------|------+                                 
                    |               +--------------------------|---------------------------|            
--------------------|-              |                          |                           |            
                    |  +------------------------+ +------------------------+  +------------------------+
                    |  |        Hippo Pod       | |        Hippo Pod       |  |       Hippo Pod        |
                    |  | +---------+ +-------+  | | +---------+ +-------+  |  | +---------+ +-------+  |
  Hippo CMS Service |  | |Hippo CMS| | Nginx |  | | |Hippo CMS| | Nginx |  |  | |Hippo CMS| | Nginx |  |
                    |  | +---------+ +-------+  | | +---------+ +-------+  |  | +---------+ +-------+  |
                    |  +------------------------+ +------------------------+  +------------------------+
                    |               |                          |                          |             
------------------------            +-----------------------------------------------------+             
                    |                                          |                                        
         Ingress    |                              +-----------------------+                            
                    |                              | Kubernetes Ingress LB |                            
                    |                              +-----------------------+                            
                                                                                    
```
As illustrated in the above diagram, overall deployment architecture invcludes,
* MySQL Service -- persietent data storage for Hippo CMS (Jacrabbit) Cluster.
* Hippo Pod contains 
  * Hippo CMS container
  * Nginx Reverse Proxy container
* Kubernetes Ingress for Load Balancing (with session affinity).

Hippo Pod
-------------
```
+------------------------------------------+    +------------------------------------------+                                
| Hippo Pod                                |    | Hippo Pod                                |                                
|                                          |    |                                          |                                
|  +----------+          +-------------+   |    |  +----------+          +-------------+   |                                
|  |Hippo CMS |          | Nginx       |   |    |  |Hippo CMS |          | Nginx       |   |                                
|  |        <-----------------         |   |    |  |        <-----------------         |   |                                
|  |     http://localhost:8080/site    |   |    |  |     http://localhost:8080/cms     |   |                                
|  +----------+          +------^------+   |    |  +----------+          +------^------+   |                                
|                               |          |    |                               |          |                                
+-------------------------------|----------+    +-------------------------------|----------+                                
                                |                                               |                                           
                                |                                               |                                           
                         site.cloud-hub.co                               cms.cloud-hub.co  
```
Kubernetes ingress is used as pure Load Balancer service, we will create a reverse proxy setup at Pod level using Nginx. Only port that is exposed from Hippo service is port 80 on nginx, Load Balancer will route traffix to Nginx, Nginx is configure to appropriately pass request to applicable context **/site** or **/cms**. This setup helps Hippo CMS Channel Manager work smoothly.
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: hippo
  labels:
    app: hippo
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: hippo
    spec:
      volumes:
      - name: hippo-conf
        configMap:
          name: hippo-conf
          items:
          - key: hippo.conf
            path: default
      - name: log
        emptyDir: {}
      containers:
      - name: hippo
        image: maheshacharya/myhippoproject-docker-deployment-demo-mysql
        ports:
        - containerPort: 8080
        restartPolicy: Always
        selector:
          app: hippo-mysql-database
      - name: nginx
        image: maheshacharya/nginx
        ports:
        - containerPort: 80
          targetPort: 80
        volumeMounts:
        - mountPath: /etc/nginx/sites-available/ 
          readOnly: true
          name: hippo-conf
        - mountPath: /var/log/nginx
          name: log
        restartPolicy: Always
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
