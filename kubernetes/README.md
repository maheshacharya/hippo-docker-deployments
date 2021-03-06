# Hippo CMS -- Deployment using Kubernetes

When it comes to Kubernetes, we talk about pods, services etc. From principles of [Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/pod/), a **Pod** is a minimum unit of deployment: a **Pod** can contain more than one container that can share the same volume, containers within the same **Pod** can communicate with other containers on using localhost address. 

Based on what we know about [Apache Jackrabbit](https://wiki.apache.org/jackrabbit/Clustering) cluster node specs (Hippo CMS uses Apache Jackrabbit repository), nodes in a cluster should have their own repository for maintaining the local repository index, therefore, a **Pod** can contain only one instance of Hippo CMS container. 

Test platform specs
-------
Single Node Kubernetes Cluster -- on AWS
* OS: Ubuntu 18.04.1 LTS x86_64 
* CPU: Intel Xeon E5-2676 v3 (4) @ 2.399GHz 
* Memory: 16039MiB 
* Kubernetes Version: v1.12.2
* Docker  Version: 18.06.1-ce


Architecture
----------
```
                                                                                                        
                    |                                                                                   
                    |                                   +-------------+                                 
   Database Service |                                   |    MySQL    |                                 
                    |                                   +------|------+                                 
                    |               +--------------------------|---------------------------|            
--------------------|               |                          |                           |            
                    |  +------------------------+ +------------------------+  +------------------------+
                    |  |        Hippo Pod       | |        Hippo Pod       |  |       Hippo Pod        |
                    |  | +---------+ +-------+  | | +---------+ +-------+  |  | +---------+ +-------+  |
  Hippo CMS Service |  | |Hippo CMS| | Nginx |  | | |Hippo CMS| | Nginx |  |  | |Hippo CMS| | Nginx |  |
                    |  | +---------+ +-------+  | | +---------+ +-------+  |  | +---------+ +-------+  |
                    |  +------------------------+ +------------------------+  +------------------------+
                    |               |                          |                          |             
--------------------|                +-----------------------------------------------------+             
                    |                                          |                                        
         Ingress    |                              +-----------------------+                            
                    |                              | Kubernetes Ingress LB |                            
                    |                              +-----------------------+                            
                                                                                    
```
As illustrated in the above diagram, overall deployment architecture includes,
* MySQL Service -- persistent data storage for Hippo CMS (Jackrabbit) Cluster.
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
Kubernetes ingress is used as pure Load Balancer service, we will create a reverse proxy setup at Pod level using Nginx. The only port that is exposed to Hippo service is port 80 on Nginx. The Load Balancer will route traffic to Nginx. Nginx is configured to appropriately pass request to applicable context **/site** or **/cms**. This setup helps the Channel Manager in CMS work without any issues.
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
Config Map
---------
[Config Map](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) is a way to manage all environmental variables subject to change per deployment. For example, a reverse proxy configuration on *test* deployment could be different from *QA* and *Production* deployments.
We are storing Nginx default.conf as a Config Map item. In this case, just change *cms.cloud-hub.co* and *site.cloud-hub.co* with an applicable server domain name.

Nginx reverse proxy example: https://www.onehippo.org/labs/configuring-nginx-as-a-reverse-proxy-for-hippo-cms.html
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: hippo-conf
data:
  hippo.conf: |
    server {
      listen       80;
      server_name "cms.cloud-hub.co";
      location / {
        # Set headers for proxy header rewriting, like ProxyPassReverse in Apache http
        # See http://wiki.nginx.org/LikeApache
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:8080/cms/;
        proxy_redirect default;
        proxy_cookie_path ~*^/.* /;
      }
      location /site/ {
        proxy_set_header Host $host;
        proxy_pass http://localhost:8080/site/;
      }
    }
    server {
      listen       80;
      server_name "site.cloud-hub.co";
      location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://localhost:8080/site/;
        proxy_redirect default;
        proxy_cookie_path ~*^/.* /;
      }
    }

```


To deploy
---------
First create a namespace, lets call it **hippo-test**
```
kubectl create namespace hippo-test
```

Next, deploy the application to namespace **hippo-test**
```
kubectl create -f hippo-mysql.yaml -n hippo-test
```
In order to access the service, you will have to install Ingress Controller and then create an ingress proxy service. 

Check for running pods
----------------------
```
kubectl get pods -n hippo-test
```
Output
```
NAME                     READY   STATUS    RESTARTS   AGE
hippo-8677b459d9-vk4w4   2/2     Running   0          38s
mysql-0                  1/1     Running   0          38s
```
Check for running services
-----------------------
```
kubectl get services -n hippo-test
```
Output
```
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hippo                  ClusterIP   10.106.52.158   <none>        80/TCP     2m34s
hippo-mysql-database   ClusterIP   10.108.6.70     <none>        3306/TCP   2m34s
```
Scaling Pods
-------
To run 4 set of replicas of Hippo Pod (Hippo CMS + Nginx) use the command below.
```
 kubectl scale  --replicas=4 deployment/hippo -n hippo-test
```
After scaling, ```kubectl get pods -n hippo-test``` should list result like below.
```
NAME                     READY   STATUS    RESTARTS   AGE
hippo-8677b459d9-4zfh6   2/2     Running   0          119s
hippo-8677b459d9-c9f2z   2/2     Running   0          119s
hippo-8677b459d9-pfhgx   2/2     Running   0          119s
hippo-8677b459d9-vk4w4   2/2     Running   0          3h56m
mysql-0                  1/1     Running   0          3h56m
```

Ingress 
--------
Why not use Node Port or Load Balancer options to expose the service? 

The challenge here is the session affinity based on a server managed entity such as JSESSIONID or ROUTE_ID, Load Balancer do offer session affinity based on client IP, which is not good for Hippo CMS, imagine yourself roaming, hopping in and out of different networks, each time your network changes, you might have new IP address and the app will prompt you for re-login -- annoying right? That is why we have to use the Ingress.

```
#Ingress for CMS
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-hash: sha1
    nginx.ingress.kubernetes.io/session-cookie-name: route
    kubernetes.io/ingress.class: nginx
  labels:
  name: cms
spec:
  rules:
  - host: cms.cloud-hub.co
    http:
      paths:
      - backend:
          serviceName: hippo
          servicePort: 80
---
#Ingress for site
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-hash: sha1
    nginx.ingress.kubernetes.io/session-cookie-name: route
    kubernetes.io/ingress.class: nginx
  labels:
  name: site
spec:
  rules:
  - host: site.cloud-hub.co
    http:
      paths:
      - backend:
          serviceName: hippo
          servicePort: 80
```
Inspect Ingress Confguration
--------
```
kubectl describe ingress cms -n hippo-test
```
Output
```
Name:             cms
Namespace:        hippo-test
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  cms.cloud-hub.co
                       hippo:80 (<none>)
Annotations:
  kubernetes.io/ingress.class:                      nginx
  nginx.ingress.kubernetes.io/affinity:             cookie
  nginx.ingress.kubernetes.io/session-cookie-hash:  sha1
  nginx.ingress.kubernetes.io/session-cookie-name:  route
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  4m17s  nginx-ingress-controller  Ingress hippo/cms                                            <none>
```


Having Trouble with Ingress configuration? 
--------------
Check out this: https://kubernetes.github.io/ingress-nginx/troubleshooting/

References
-------
* Get started with Kubernetes: https://kubernetes.io/docs/
* Install Kubernetes Cluster using Kubeadm: https://kubernetes.io/docs/setup/independent/install-kubeadm/
* Kubernetes Ingress Controller: https://kubernetes.github.io/ingress-nginx/deploy/
