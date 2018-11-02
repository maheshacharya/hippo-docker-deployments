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
Deploy application
```
cd kubernetes
oc create -f hippo-mysql.yaml -n hippo
```

Create a Route for Hippo Service
------
TBD
