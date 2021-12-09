# Chapter 5. Customizing Source-to-Image Builds.



**Abstract**



| **Goal**       | Customize an existing S2I builder image and create a new one. |
| -------------- | ------------------------------------------------------------ |
| **Objectives** | Describe the required and optional steps in the Source-to-Image build process.                                            Customize an existing S2I builder image with scripts.                                          Create a new S2I builder image with S2I tools. |
| **Sections**   | Describing the Source-to-Image Architecture (and Quiz)                                          Customizing an Existing S2I Builder Image (and Guided Exercise)                                          Creating an S2I Base Image (and Guided Exercise) |
| **Lab**        | Customizing Source-to-Image Builds                           |

## Describing the Source-to-Image Architecture

### Source-to-Image (S2I) Language Detection

Openshift kan lave applikationer ud fra source kode gemt i et git repository. Den simpleste form af `oc new app` kommandoen kræver kun et github repo som kommando. 

| Files                     | Language builder | Programming language       |
| ------------------------- | ---------------- | -------------------------- |
| `Dockerfile`              | N/A              | Dockerfile build (not S2I) |
| `pom.xml`                 | `jee`            | Java (with JBoss EAP)      |
| `app.json,package.json`   | `nodejs`         | Node.js (JavaScript)       |
| `composer.json,index.php` | `php`            | PHP                        |

RHOCP følger en multistep algorithme til at afgøre om en url peger på et source kode repository og hvis 

1. RHOCP accesses the URL as a container registry URL. If this succeeds, there  is no need to create a build configuration. RHOCP creates the deployment and other resources required to deploy a container image.
2. If the URL points to a Git repository, RHOCP retrieves a file list from the repository and searches for a `Dockerfile` file. If found, the build configuration uses a `docker` strategy. Otherwise, the build configuration uses a `source` strategy, which needs an S2I builder image.
3. RHOCP searches for image streams that match the language builder name as the value of the `supports` annotation. The first match becomes the S2I builder image.
4. If no annotation matches, RHOCP searches for an image stream whose name  matches the language builder name. The first match becomes the S2I  builder image.

Referer til et image registry

```bash
oc new-app registry.access.redhat.com/ubi8/ubi:8.0
```

Referer til git

```bash
oc new-app  https://github.com/RedHatTraining/DO288-apps/ubi-echo
```

Brug en bestemt image stream.

```bash
oc new-app -i php:7.3 https://github.com/RedHatTraining/DO288-apps/php-helloworld
```

## S2I build process

**Application source code**

Source kode for applikationen

**S2I scripts**

En række scripts som Openshift eksekverer for at customizere S2I builder imaget. S2I scripts kan skrives i et vilkårligt programmerings sprog, så længe at disse scripts er eksekverbare indeni S2I builder imaget

**The S2I builder image**



The S2I build process relies on some S2I scripts, which it executes at  various stages of the build workflow. The scripts, and a brief  description of what they do, are listed in the following table:

| Script           | Mandatory? | Description                                                  |
| ---------------- | ---------- | ------------------------------------------------------------ |
| `assemble`       | Yes        | The `assemble` script builds the application from source and places it into appropriate directories inside the image. |
| `run`            | Yes        | The `run` script executes your application. It is recommended to use the `exec` command when running any container processes in your `run` script. This ensures signal propagation and graceful shutdown of any process launched by the `run` script. |
| `save-artifacts` | No         | The `save-artifacts` script gathers all dependencies required for the application and saves  them to a tar file to speed up subsequent builds. For example, for Ruby, gems installed by Bundler, or for Java, .m2  contents. This means that the build does not have to redownload these  contents, improving build time. These dependencies are gathered into a  tar file and streamed to the standard output. |
| `usage`          | No         | The `usage` script provides a description of how to properly use your image. |
| `test/run`       | No         | The `test/run` script allows you to create a simple process to verify if the image is working correctly. |

## S2I build workflow

1. RHOCP instantierer en container baseret på et S2I builder image og raver en tar fil der indeholder source koden og S2I scripts. Derefter streamer RHOCP tar filen ind i containeren
2. Inden assemble scriptet kører RHOCP ekstraherer tar filen og gemmer indholdet ind i en lokation specificeret enten af --destination optionen eller af io.openshift.s2i.destination label'et fra builder imaget. Default location er /tmp mappen
3. Hvis det er et incrementalt buil, restorer assemble scriptet artefakterne der tidligere var gemt i tar filen af save-artefact scriptet
4. assemble scriptet bygger applikationen fra source og placerer binaries ind i de korrekte mapper
5. Hvis dette er et incrementalt build, save-artifact scriptet eksekveres og gemmer alle dependency build artifakter til en tar fil.
6. Efter assemble scriptet er færdigt, bliver containeren comittet for at lave det endelige image og RHOCP sætter run scriptet som CMD instruktionen for det endelige image.

![](/Users/jacobbennedsen/Projects/Learning/Redhat/DO288/resources/s2i-workflow.svg)

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi:8.0


LABEL io.k8s.description="My custom Builder" \ 
    io.k8s.display-name="Nginx 1.6.3" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,webserver,html,nginx" \
    io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" 

RUN yum install -y epel-release && \ 
    yum install -y --nodocs nginx && \
    yum clean all

EXPOSE 8080

COPY ./s2i/bin/ /usr/libexec/s2i
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#customizing-s2i-s2i-lecture-CO1-1) | Set the labels that are used for RHOCP to describe the builder image. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#customizing-s2i-s2i-lecture-CO1-2) | Tell S2I where to find its mandatory scripts (`run`, `assemble`). |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#customizing-s2i-s2i-lecture-CO1-3) | Install the NGINX web server package and clean the Yum cache. |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#customizing-s2i-s2i-lecture-CO1-4) | Set the default port for applications built using this image. |
| [![5](https://rol.redhat.com/rol/static/roc/Common_Content/images/5.svg)](https://rol.redhat.com/rol/app/#customizing-s2i-s2i-lecture-CO1-5) | Copy the S2I scripts to the `/usr/libexec/s2i` directory.    |

The assemble script may be defined as follows:

```bash
#!/bin/bash -e

if [ "$(ls -A /tmp/src)" ]; then
  mv /tmp/src/* /usr/share/nginx/html/
fi
```

Override the default NGINX `index.html` file.

```bash
#!/bin/bash -e
/usr/sbin/nginx -g "daemon off;"
```

Prevent the NGINX process from running as a daemon so that so that the container does not exit after the process runs `exec` script.

#### Overriding S2I Builder Scripts

Det er muligt at levere sine egne builder scripts, ved at placere dem i s2i/bin mappen i dit repository

Det er også muligt at lave en wrapper ved at downloade scripts'ne og derefter lave dit eget script der extender disse.

S2I builder images provide default S2I scripts. You can override the  default S2I scripts to change how your application is built and  executed. You can override the default build behavior without needing to create a new S2I builder image by forking the source code for the  original S2I builder.

The simplest way to  override the default S2I scripts for an application is to include your  S2I scripts in the source code repository for your application. You can  provide S2I scripts in the `.s2i/bin` folder of the application source code repository.

When RHOCP starts the S2I process, it inspects the source code folder, the  custom S2I scripts, and the application source code. RHOCP includes all  of these files in the tar file injected into the S2I builder image.  RHOCP then executes the custom `assemble` script instead of the default `assemble` script included with the S2I builder, followed by the other overridden custom scripts (if any).

# Customizing an Existing S2I Base Image

### Objectives

After completing this section, you should be able to customize the S2I scripts of an existing S2I builder image.

## Customizing Scripts of an S2I Builder Image

Brug podman pull for at hente et image ned

```bash
 podman pull myregistry.example.com/rhscl/php-73-rhel7
```

Og derefter kan du finde lokationen af scriptet

```bash
podman inspect \
--format='{{ index .Config.Labels "io.openshift.s2i.scripts-url"}}' \
rhscl/php-73-rhel7
```

Du kan også bruge skopeo

```bash
skopeo inspect docker://myregistry.example.com/rhscl/php-73-rhel7 | grep io.openshift.s2i.scripts-url
```

Du kan for eksempel lave en wrapper for assemble scriptet in .s2i/bin folderen

```bash
#!/bin/bash
echo "Making pre-invocation changes..."

/usr/libexec/s2i/assemble
rc=$?

if [ $rc -eq 0 ]; then
    echo "Making post-invocation changes..."
else
    echo "Error: assemble failed!"
fi

exit $rc
```

```bash
#!/bin/bash
echo "Before calling run..."
exec /usr/libexec/s2i/run
```

## Incremental Builds in S2I

*The S2I build process provides a mechanism to reduce build time by invoking the `save-artifacts` script after invoking the `assemble` script, as part of the S2I life cycle. The `save-artifacts` script ensures that dependent artifacts (libraries and components required for the application) are saved for future builds.*

During the next build, the `assemble` script restores the cached artifacts before building the application from source code. Note that the `save-artifacts` script is responsible for streaming dependencies to stdout in a tar file.

The `save-artifacts` script that caches Maven JAR files can be defined as follows:

```bash
#!/bin/sh -e

# Stream the .m2 folder tar archive to stdout
if [ -d ${HOME}/.m2 ]; then
    pushd ${HOME} > /dev/null
    tar cf - .m2
    popd > /dev/null
fi
```

```bash
# Restore the .m2 folder
...output omitted...
if [ -d /tmp/artifacts/.m2 ]; then
  echo "---> Restoring maven artifacts..."
  mv /tmp/artifacts/.m2 ${HOME}/
fi
...output omitted...
```

# Creating an S2I Base Image

### Objectives

After completing this section, you should be able to create an S2I builder image using the `s2i` command-line tool.