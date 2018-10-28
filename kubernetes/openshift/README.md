# Hippo CMS -- OpenShift Deployment
Kubetnetes is great, but OpenShift makes Kubernetes even simpler and manageable especially when we talk about managing 100's of pods and services.

To Deploy 
---------
Create new project on your OpenShift cluster.
```
oc new-project hippo
```
Deploy application
```
cd kubernetes
oc create -f hippo-mysql.yml -n hippo
```

Create a Route for Hippo Service
------
TBD
