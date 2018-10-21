Deploy Hippo CMS using Docker Compose
=====================================
Read more on Docker Compose documentation here: https://docs.docker.com/compose/

Apply Configuration Changes
---------------------------
Modify *default* and *default-ssl* file under apache2/sites-available to add proper server/domain name:
Replace *<domain name>* with server DNS name, for example: example.com, cms.example.com etc.


Deploy Hippo CMS
----------
Change current working directory to docker-compose and then run:
```
$ docker-compose up -d
```
