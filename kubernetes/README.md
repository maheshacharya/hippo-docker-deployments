# Hippo CMS -- Deployment using Kubernetes

To deploy
---------
First create a namespace, lets call it **hippo-test**
```
kubectl create -ns hippo-test
```

Next, deploy the application
```
kubectl create -f hippo-deployment.yam -n hippo-test
```
In order to access the service, you will have to install Ingress Controller and then create an ingress proxy service. 
