# Hippo CMS -- Deployment using Kubernetes

```
kubectl create -ns hippo-test
kubectl create -f hippo-deployment.yam -n hippo-test
```
In order to access the service, you will have to install Ingress Controller and then create an ingress proxy service. 
