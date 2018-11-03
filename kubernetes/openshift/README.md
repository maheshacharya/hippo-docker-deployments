# Hippo CMS -- OpenShift Deployment
Kubetnetes is great, but OpenShift makes Kubernetes even simpler and manageable especially when we talk about managing 100's of pods and services. OpenShift enables deployment through web application console, however, we will be using 'oc' commmand to perform deployment process.

How to install OpenShift Origin on Ubuntu 18.04?
-----------------------------------------------
Checkout this article: 
https://medium.com/@maheshacharya_44641/install-openshift-origin-on-ubuntu-18-04-7b98773c2ee6

To Deploy 
---------
Create new project on your OpenShift cluster.
```
oc new-project hippo
```
Enable security context for non-root user for applications
-----
without this you will run into permission [issue](https://github.com/openshift/origin/issues/10483) 
```
docker exec origin oc adm policy add-scc-to-user anyuid -z default -n hippo
```

Deploy application
```
cd kubernetes
oc create -f hippo-mysql.yaml -n hippo
```
Output
-----
```
configmap "hippo-conf" created
deployment "hippo" created
statefulset "mysql" created
service "hippo" created
service "hippo-mysql-database" created
```
Check for running pods
--------
```
oc get pods
```
Output
Basically, we have two pods running 
* Hippo (Pod) -- has 2 containers
  * Hippo CMS (Container)
  * Nginx Reverse Proxy (Container)
* MySQL (Pod) -- has 1 container
```
NAME                    READY     STATUS    RESTARTS   AGE
hippo-7dc69df6d-zxtcq   2/2       Running   0          25s
mysql-0                 1/1       Running   0          25s
```
Check for services
--------
```
oc get services
```
Output
```
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
hippo                  ClusterIP   172.30.220.182   <none>        80/TCP     2m
hippo-mysql-database   ClusterIP   172.30.222.113   <none>        3306/TCP   2m
```

Create a Route for Hippo Service
------
Change directory to 'openshift' and run following commands. 
Please note: change "site.cloud-hub.co" and "cms.cloud-hub.co" domain name with your own domain names for site and cms. 
```
oc create -f site-route.yaml -f  cms-route.yaml -n hippo
```
Output
```
route "site" created
route "cms" created
```
Check for routes
```
oc get routes
```
Output
```
NAME      HOST/PORT           PATH      SERVICES   PORT      TERMINATION   WILDCARD
cms       cms.cloud-hub.co              hippo      80                      None
site      site.cloud-hub.co             hippo      80                      None
```
Scaling Hippo Pods
-------
Lets run 4 instances of Hippo pods -- which means, we will be running 4 Hippo CMS instances.
```
oc scale  --replicas=4 deployment/hippo -n hippo
```
```oc get pods``` command will show following results
```
hippo-7dc69df6d-fzpdb   2/2       Running   0          9s
hippo-7dc69df6d-m7bwk   2/2       Running   0          9s
hippo-7dc69df6d-wknw8   2/2       Running   0          9s
hippo-7dc69df6d-zxtcq   2/2       Running   0          34m
mysql-0                 1/1       Running   0          34m
```


