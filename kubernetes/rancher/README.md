# Hippo CMS -- Rancher Deployment

**This doc is still being developed, stay tuned..**


Install Rancher
---------------
There multiple ways to install and start rancher depending on whether you are running  a test instance or a production cluster. Follow Rancher's own guidelines here:
https://rancher.com/docs/rancher/v2.x/en/installation/

For our test installation, we ran this command below:

docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /host/rancher:/var/lib/rancher \
  rancher/rancher:latest

The above command will install Rancher with persitent data volume on the host system.

Add a Cluster
-------------
* Directions on how to setup a cluster is not discussed here, pleaes follow guidlines on Rancher documentation.
* We will be selecting Aamazon EC2 for infrastructure provider
* We have spun up a cluster with 3 worker nodes. 
* Make sure that the rancher host and worker nodes are in the same availability zone, otherwise you will run into frequent application session expiration due to difference in server times. 
* It takes about 5-10 minutes for cluster to be prepared and ready for use.

Application Deployment
---------------------
Once the cluster is ready, copy the kubectl.config to your local machine or run commands directly on Rancher terminal. 

create a hippo-mysql.yml with content from this file.
https://github.com/maheshacharya/hippo-docker-deployments/blob/master/kubernetes/hippo-mysql.yaml

Run hippo-mysql.yaml

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

```
> kubectl get pods -n hippo
NAME                    READY     STATUS    RESTARTS   AGE
hippo-58b8d5b67-j6tfn   2/2       Running   0          47s
mysql-0                 1/1       Running   0          47s

```
Check all running services in namespace 'hippo'

```

> kubectl get services -n hippo
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
hippo                  ClusterIP   10.43.250.54    <none>        80/TCP     1m
hippo-mysql-database   ClusterIP   10.43.191.101   <none>        3306/TCP   1m

```

In the rancher UI move 'hippo' namespace under a new project "hippo-demo"

```
> kubectl create -f hippo-ingress.yaml -n hippo
ingress.extensions "cms" created
ingress.extensions "site" created
```




