# Chapter 8. Building Applications for OpenShift

## Defining External Services

Openshift har flere forskellige måder hvorpå en ekstrern service defineres. Dette tillader openshift at pege til en eller flere hosts udenfor et openshift cluster. 

Se en applikation som du ønsker at containerizere der er afhængig af en eksisterende database service der ikke er tilgængelig direkte indeni dit openshift cluster. Det er ikke nødvendigt at migrere databasen, i stedet kan du designe din applikation til at interagere med openshift services og database servicen.

### Create an external service

```bash
oc create service externalname myservice \
--external-name myhost.example.com
```

## Defining endpoints for a service

A typical service creates *endpoint* resources dynamically, based on the selector attribute of the service. The `oc status` and `oc get all` commands do not display these resources. You can use the `oc get endpoints` command to display them.

If you use the `oc create service externalname --external-name` command to create a service, then the command also creates an endpoint  resource that points to the host name or IP address given as an  argument.

If you do not use the `--external-name` option, it does not create an endpoint resource. In this case, use the `oc create -f` command and a resource definition file to explicitly create the endpoint resources.

If you create an endpoint from a file, then you can define multiple IP  addresses for the same external service, and rely on the OpenShift  service load-balancing features. In this scenario, OpenShift does not  add or remove addresses to account for the availability of each  instance. An external application must update the list of IP addresses  in the endpoint resource.

See the references at the end of this section for more information about endpoint resource definition files.

# Containerizing Services

Det at en applikation allerede er containerized af en vendor gør ikke nødvendigvis at det kører godt på openshift. Dit første forsøg på at deploye denne applikation via `oc new-app` kommandoen fejler muligvis for flere mulige grunde. At løse dette problem kræver muligvis customizering af applikationens deployment eller ændringer til dens dockgrfil.

The following is a non-exhaustive list of common issues:

- Your application does not run under the default OpenShift security policies defined by the `retricted` Security Context Constraint (SCC). The application's container image might expect to run as the `root` or another fixed user. It might also require custom SELinux policies and other privileged settings.

  Usually, you can change the Dockerfile so that the application complies with the default OpenShift security policies. If not, then carefully evaluate  the risk of running the application under a more relaxed security  policy.

- Your application requires custom settings for resource requests and resource limits. This happens with  legacy, monolithic applications, that were not designed for a  microservice-based architecture. OpenShift cluster administrators set up default limit ranges that provide default values for resource requests  and limits to your project, and these might be too small for your legacy application.

  You can specify increased resource requests and  resource limits on your deployment to override the defaults set by the  cluster administrator. If your application requirements are over the  resource quota set by your OpenShift cluster administrators, the  administrators must either increase the resource quota for your user, or provide a service account for your application with an increased  resource quota.

- Your application image  defines configuration settings that are fine for a standalone container  runtime, but not for an OpenShift or a generic Kubernetes cluster.  Applications usually read configuration settings from environment  variables, and define defaults by using `ENV` instructions in a Dockerfile.

  There is no need to change the Dockerfile to change its environment  variables. You can override any environment variable, including those  that are not explicitly set by `ENV` instructions in the Dockerfile, on your application deployment.

- Your application requires persistent storage. Applications usually define storage requirements using `VOLUME` instructions in the Dockerfile. Sometimes these applications expect to  use host folders for their volumes, which is forbidden by your OpenShift cluster administrators. These applications might expect to have direct  access to network storage, which is also not recommended from a security standpoint.

  The proper way to provide persistent storage for  applications on OpenShift is to define persistent volume claim (PVC)  resources, and to configure the application deployment to attach these  PVCs to the volumes from the application container. OpenShift manages  network storage on behalf of your applications, and your cluster  administrators manage security policies and performance levels for  storage.

Din vender giver muligvis kubernetes resource filer for at deploye applikationer og disse resourcer antaget at brugeren er administrator. Disse vendors overvejer muligvis ikke at et openshift cluster deles mellem flere organisationer og teams. 

## Reviewing the dockerfile for the nexus applikation.

Nexus applikationen udviklet af Sonatype er en repository manager der oftest er brugt af udviklere for at gemme artefakter krævet af applikationer såsom dependencies. Denne applikation kan gemme artifakter for adskillige teknologier såsom Java, .NET, Docker, Node, etc. Sonatype giver en dockerfile for at bygge nexus applikationen ved brug af et base container image fra redhat. 

The resulting image is available from the Red Hat Container Catalog,  which attests that it complies with a core set of guidelines from  Red Hat. The Dockerfile is configured to support OpenShift features,  such as allowing container images to run as a random user account.

### Base image and container metadata

```
...output omitted...
FROM       registry.access.redhat.com/rhel7/rhel 

MAINTAINER Sonatype <cloud-ops@sonatype.com>
...output omitted...

LABEL name="Nexus Repository Manager" \
      ...output omitted...
      io.k8s.description="The Nexus Repository Manager server \
          with universal support for popular component formats." \ 
      io.k8s.display-name="Nexus Repository Manager" \
      io.openshift.expose-services="8081:8081" \
      io.openshift.tags="Sonatype,Nexus,Repository Manager"

...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO1-1) | Specifies the Red Hat Enterprise Linux 7 container image as the base image. There is also an alternative `Dockerfile` file that uses the Red Hat Universal Base Image 8 located in the GitHub project. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO1-2) | Defines the image metadata for Kubernetes and OpenShift.     |

The Nexus Dockerfile also uses the Dockerfile `ARG` instruction. An `ARG` instruction defines a variable that only exists during the image build process. Unlike an `ENV` instruction, these variables are not a part of the resulting container image:

```
...output omitted...
ARG NEXUS_VERSION=3.18.0-01
ARG NEXUS_DOWNLOAD_URL=https://.../nexus/3/nexus-${NEXUS_VERSION}-unix.tar.gz
ARG NEXUS_DOWNLOAD_SHA256_HASH=e1d9...c99e
...output omitted...
```

The build process uses these three variables to control and verify the installed version of Nexus in the container image.

The Dockerfile also defines environment variables that are present in the built container image:

```
...output omitted...
# configure nexus runtime
ENV SONATYPE_DIR=/opt/sonatype
ENV NEXUS_HOME=${SONATYPE_DIR}/nexus \
    NEXUS_DATA=/nexus-data \
    NEXUS_CONTEXT='' \
    SONATYPE_WORK=${SONATYPE_DIR}/sonatype-work \
DOCKER_TYPE='rh-docker'
...output omitted...
```

The installation process uses the `DOCKER_TYPE` environment variable to customize installation and configuration. A value of `rh-docker` invokes any configuration changes necessary for a Red Hat certified container image.

### Nexus intallation process.

Sonatype maintains Chef *cookbooks* that standardize the installation of the Nexus application. Chef is an  open source configuration management technology, similar to Puppet and  Ansible, that aims to streamline the process of configuring and managing servers. A Chef cookbook is a collection of Chef *recipes*, and each recipe defines the configuration for a particular set of system resources.

The Nexus Dockerfile uses the Chef installation process to build the Nexus container image:

### Customizing OpenShift Resources for the Nexus Image

If you build a Nexus container image from its official Dockerfile, and deploy it using a command such as `oc new-app --docker-image`, it might not work reliably. To ensure the container image works well, you must make a few customizations to its deployment.

#### Setting Resource Requests and Limits

Red Hat recommends that applications deployed to OpenShift define resource  requests and resource limits. If you are not using very old releases of  the Java Virtual Machine (JVM), you do not need to define JVM heap  options.

Resource requests state the minimum amount of resources the application requires to run. The OpenShift  scheduler finds a worker node in the cluster with sufficient available  CPU and memory to run the application, which prevents resource  starvation failures at start up.

Resource Limits state how much  the CPU and memory usage of an application can increase over time.  OpenShift sets Linux kernel cgroups for the application container, which prevents it from ever violating those limits. The Linux kernel kills  containers that violate these limits, and OpenShift starts replacement  containers, thus mitigating issues caused by memory leaks, infinite  loops, and deadlocks. Recent releases of the JVM automatically size the  heap and other internal data structures to not violate current cgroups  settings.

You define resource requests and resource limits inside the pod template of a deployment

You define resource requests and resource limits inside the pod template of a deployment:

```
...output omitted...
- apiVersion: apps/v1
  kind: Deployment
...output omitted...
  spec:
...output omitted...
    template:
...output omitted...
      spec:
        containers:
...output omitted...
          resources:
            limits:
              cpu: "1"
              memory: 2Gi
            requests:
              cpu: 500m
              memory: 256Mi
...output omitted...
```

In the Nexus scenario, you must also override the `INSTALL4J_ADD_VM_PARAMS` environment variable to remove any JVM options related to memory sizing:

```
...output omitted...
- apiVersion: apps/v1
  kind: Deployment
...output omitted...
  spec:
...output omitted...
    template:
...output omitted...
      spec:
        containers:
        - env:
            - name: INSTALL4J_ADD_VM_PARAMS
              value: -Djava.util.prefs.userRoot=/nexus-data/javaprefs
...output omitted...
```

The preceding example assumes that you are not overriding the `NEXUS_DATA` environment variable to specify a different volume path for the container's persistent storage.

#### Implementing Probes

```
#!/bin/sh
curl -si -u admin:$(cat /nexus-data/admin.password) \ 
  http://localhost:8081/service/metrics/ping | grep pong 
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO4-1) | The `-u` option passes a username and password in the HTTP GET request. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO4-2) | Nexus provides the `/service/metrics/ping` endpoint, which must return the `pong` value when the service is ready. If `pong` is not in the response, the `grep` command exits with a nonzero status code. |

In a similar way, the following script determines if the Nexus service is alive:

```
#!/bin/sh
curl -si -u admin:$(cat /nexus-data/admin.password) \
  http://localhost:8081/service/metrics/healthcheck | grep healthy | \
  grep true
```

Because these probe scripts are simple, you can put  the content of each script in the appropriate probe stanza of the Nexus  container specification:

```
...output omitted...
- apiVersion: apps/v1
  kind: Deployment
...output omitted...
  spec:
...output omitted...
    template:
...output omitted...
      spec:
        containers:
...output omitted...
          livenessProbe:
            exec:
              command:
              - /bin/sh 
              - "-c" 
              - > 
               curl -si -u admin:$(cat /nexus-data/admin.password)
               http://localhost:8081/service/metrics/healthcheck |
               grep healthy | grep true
            failureThreshold: 3
            initialDelaySeconds: 10
            periodSeconds: 10
...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO5-1) | Use the `/bin/sh` shell.                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO5-2) | The `-c` option indicates that a single command is executed in the shell. |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#migrating-containerizing-lecture-CO5-3) | The YAML multiline continuation indicator. This allows you to split a long string over several lines. |

For a large probe script, consider modifying the Dockerfile and adding the probe scripts to the container image.

# Deploying Cloud-Native Applications with JKube

