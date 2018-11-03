# Hippo CMS -- OpenShift Deployment
Kubetnetes is great, but OpenShift makes Kubernetes even simpler and manageable especially when we talk about managing 100's of pods and services.

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
```
NAME                    READY     STATUS    RESTARTS   AGE
hippo-7dc69df6d-zxtcq   2/2       Running   0          25s
mysql-0                 1/1       Running   0          25s
```

Create a Route for Hippo Service
------
TBD
