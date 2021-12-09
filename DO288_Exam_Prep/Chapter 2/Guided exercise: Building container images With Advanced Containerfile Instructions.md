# Building Container Images with Advanced Containerfile Instructions

## Formål med øvelsen



* Lav en Apache HTTP server container image ved at bruge en containerfil og deploy den til et openshift cluster.
* Lav et child container image ved at extende parent HTTP server imaget
* Ændre containerfilen for en child container image så det køre på et openshift cluster med et random userid.

## 1 - Se containerfilen

A prebuilt Apache HTTP Server parent container image is provided in the Quay.io public registry at `quay.io/redhattraining/httpd-parent`. Briefly review the Containerfile for this parent image located at `~/DO288/labs/container-build/httpd-parent/Containerfile`:

```
FROM registry.access.redhat.com/ubi8/ubi:8.0 

MAINTAINER Red Hat Training <training@redhat.com>

# DocumentRoot for Apache
ENV DOCROOT=/var/www/html 

RUN   yum install -y --no-docs --disableplugin=subscription-manager httpd && \ 
      yum clean all --disableplugin=subscription-manager -y && \
      echo "Hello from the httpd-parent container!" > ${DOCROOT}/index.html

# Allows child images to inject their own content into DocumentRoot
ONBUILD COPY src/ ${DOCROOT}/ 

EXPOSE 80

# This stuff is needed to ensure a clean start
RUN rm -rf /run/httpd && mkdir /run/httpd

# Run as the root user
USER root 

# Launch httpd
CMD /usr/sbin/httpd -DFOREGROUND
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO1-1) | The base image is the Universal Base Image (UBI) for Red Hat Enterprise Linux 8.0 from the Red Hat Container Catalog. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO1-2) | Environment variables for this container image.              |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO1-3) | The `RUN` instruction contains several commands that install the Apache HTTP Server and create a default home page for the web server. |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO1-4) | The `ONBUILD` instruction allows child images to provide their own customized web  server content when building an image that extends from this parent  image. |
| [![5](https://rol.redhat.com/rol/static/roc/Common_Content/images/5.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO1-5) | The `USER` instruction runs the Apache HTTP Server process as the `root` user. |

## 2 - extend base imaget

```bash
source /usr/local/etc/ocp4.config
```

```bash
git checkout main
```

```bash
git checkout -b container-build
```

```bash
git push -u origin container-build
```

### Inspicer imaget i DO288-apps/container-build/Containerfile

Filen indeholder en enkelt linje

```dockerfile
FROM quay.io/redhattraining/http-parent
```

Child imaget har sin egen Index.html fil der erstatter parent containerens index.html fil

```bash
<!DOCTYPE html>
<html>
<body>
  Hello from the Apache child container!
</body>
</html>
```

Login

```bash
oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

Lav et nyt projekt

```bash
oc new-project ${RHT_OCP4_DEV_USER}-container-build
```

Byg HTTP server imaget

```bash
podman build --layers=false -t do288-apache ./container-build
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO2-1) | The Containerfile is automatically identified and podman pull the parent image. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO2-2) | The `ONBUILD` instruction in the parent Containerfile triggers the copying of the child's `index.html` file, which overwrites the parent index file. |

Se dine images

```bash
podman images
```

Tag og push imaget til Quay

```bash
podman tag do288-apache quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

```bash
podman login quay.io -u ${RHT_OCP4_QUAY_USER}
```

```bash
podman push quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

Deploy Apache HTTP serveren fra Quay.io til openshift clusteret

```bash
oc new-app --name hola quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

Brug oc get pods for at se at builded fejler

```bash
oc get pods
```

Og brug derefter oc logs for at se hvad der er galt.

```
[student@workstation DO288-apps]$ oc logs hola-13p75f5
AH00558: httpd: Could not reliably determine the server's fully qualified domain name...
(13)Permission denied: AH00072: make_sock: could not bind to address [::]:80 
(13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:80 
no listening sockets available, shutting down
AH00015: Unable to open logs 
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO3-1) | Because OpenShift runs containers using a random userid, ports below 1024 are privileged and can only be run as `root`. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#designing-dockerfile-ge-practice-CO3-2) | The random userid used by OpenShift to run the container does not have permissions to read and write log files in `/var/log/httpd` (the default log file location for the Apache HTTP Server on RHEL 7). |

Slet alle resourcer fra openshift projektet

```bash
oc delete all -l app=hola
```

Rediger containerfilen i /DO288-apps/container-build/Containerfile og gør følgende

Override the `EXPOSE` instruction from the parent image and change the port to 8080.

```
EXPOSE 8080
```

```
LABEL io.openshift.expose-services="8080:http"
```

Update the list of labels to include the `io.k8s.description`, `io.k8s.display-name`, and `io.openshift.tags` labels that OpenShift consumes to provide helpful metadata about the container image:

```
LABEL io.k8s.description="A basic Apache HTTP Server child image, uses ONBUILD" \
      io.k8s.display-name="Apache HTTP Server" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="apache, httpd"
```

You need to run the web server on an unprivileged port (that is, greater than 1024). Use a `RUN` instruction to change the port number in the Apache HTTP Server configuration file from the default port 80 to 8080:

```
RUN sed -i "s/Listen 80/Listen 8080/g" /etc/httpd/conf/httpd.conf
RUN sed -i "s/#ServerName www.example.com:80/ServerName 0.0.0.0:8080/g" /etc/httpd/conf/httpd.conf
```

Change the group ID and permissions of the folders where the web server process reads and writes files:

```
RUN chgrp -R 0 /var/log/httpd /var/run/httpd && \
    chmod -R g=u /var/log/httpd /var/run/httpd
```

Add a `USER` instruction for an unprivileged user. The Red Hat convention is to use userid 1001:

```
USER 1001
```

Save the Containerfile and commit the changes to the Git repository from the `~/DO288-apps/container-build` folder:

```
[student@workstation DO288-apps]$ cd container-build
[student@workstation container-build]$ git commit -a -m \
"Changed the Containerfile to enable running as a random uid on OpenShift"
...output omitted...
[student@workstation container-build]$ git push
...output omitted...
[student@workstation container-build]$ cd ..
```

Fjern gamle images

```bash
podman rmi -a --force
```

Byg imaged

```bash
podman build --layers=false -t do288-apache ./container-build
```

Tag imaget

```
podman tag do288-apache quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

Push til quay

```
podman push quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

Redeploy image

```
oc new-app --name hola quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

Se pods

```
oc get pods
```

Expose servicen

```
oc expose --port='8080' svc/hola
```

Se route

```
oc get route
```

Slet projekt

```
oc delete project ${RHT_OCP4_DEV_USER}-container-build
```

```
lab container-build finish
```