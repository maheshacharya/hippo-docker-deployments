# Hippo CMS -- Deployment using Docker Swarm
Docker Swarm is an elegant platform for deploying a highly-available  and scalable cluster of Hippo CMS application. You can run 100's of containers of Hippo CMS instances at runtime, scale-up or scale-down based on the processing requirements. You can scale underlying infrastructure horizontally by adding new resoures (VM nodes). 

Goals
-----
* Use MySQL persistence storage for the repository.
* Start the cluster with 4 containers of Hippo CMS.
* Use [Traefik](https://traefik.io/) load balancer to acheive JSESSIONID based session affinity.
* Use Apache(http) reverse proxy.

Deploy the stack
-----------------
```
cd docker-swarm
docker swarm init
docker stack deploy -c docker-compose.yml hippo
```
