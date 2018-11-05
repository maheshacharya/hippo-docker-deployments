# Hippo CMS -- Rancher Deployment

Rancher 2.0 is a Container Orchestration platform that is built on top of Kubernetes. Rancher provides UI and command line tooling for provisioning and managing a large number of clusters. I have used rancher for about 2-3 years now, and it's a cool Open Source solution when it comes to Kubernetes. 

Our Environment Specs
-------------
* OS: Ubuntu 18.04.1 LTS x86_64 -- Rancher Host Machine
* CPU: Intel Xeon E5-2676 v3 (4) @ 2.399GHz
* Memory: 16039MiB
* Docker Version: 18.06.1-ce
* Rancher Version: v2.1.1
* Rancher Cluster Worker Nodes (3) 
  * AWS t2.large instance with 50GB storage.

Install Rancher 2.0
---------------
There are multiple ways to install and start rancher -- all depends on whether you are running a test instance or a production cluster. Follow Rancher's own guidelines here:
https://rancher.com/docs/rancher/v2.x/en/installation/

For our test installation, we ran this command below:
```
  docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /host/rancher:/var/lib/rancher \
  rancher/rancher:latest
``` 

The above command will install Rancher with persistent data volume on the host system.

Add a Cluster
-------------
* Directions on how to set up a cluster is not discussed here, please follow guidelines on [Rancher documentation](https://rancher.com/docs/rancher/v2.x/en/installation/).
* We have selected Amazon EC2 for infrastructure provider.
* For our test, we have spun up a cluster with 3 worker nodes. 
* Make sure that the rancher host and worker nodes are in the same availability zone, otherwise you will run into frequent application session expiration due to the difference in server times. 
* It takes about 2-4 minutes for the cluster to be prepared and ready for use.

Application Deployment
---------------------
Once the cluster is ready, copy the kubectl.config to your local machine or run commands directly on Rancher terminal. 

create a hippo-mysql.yml with content from this file:
* https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/hippo-mysql.yaml

Deploy hippo-mysql.yaml

Create namespace 'hippo'

> kubectl create namespace hippo

Deploy 

> kubectl create -f hippo-mysql.yaml -n hippo

Output

configmap "hippo-conf" created
deployment.extensions "hippo" created
statefulset.apps "mysql" created
service "hippo" created
service "hippo-mysql-database" created


> kubectl get pods -n hippo
```
NAME                    READY     STATUS    RESTARTS   AGE
hippo-58b8d5b67-j6tfn   2/2       Running   0          47s
mysql-0                 1/1       Running   0          47s

```
Check all running services in namespace 'hippo'

> kubectl get services -n hippo

```
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hippo                  ClusterIP   10.43.250.54    <none>        80/TCP     1m
hippo-mysql-database   ClusterIP   10.43.191.101   <none>        3306/TCP   1m
```

In the rancher UI move 'hippo' namespace under a new project "hippo-demo"

Deploy [ingress](https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/rancher/hippo-ingress.yaml) with session affinity enabled.


> kubectl create -f hippo-ingress.yaml -n hippo

```
ingress.extensions "cms" created
ingress.extensions "site" created
```



References
-------
* [Nginx Ingress Annotations](https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md)
* [Rancher Documentation](https://rancher.com/docs/rancher/v2.x/en/)
* [Rancher Youtube Channel -- Great Learning Videos](https://www.youtube.com/channel/UCh5Xtp82q8wjijP8npkVTBA)

