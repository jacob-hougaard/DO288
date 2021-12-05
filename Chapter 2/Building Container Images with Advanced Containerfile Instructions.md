# Building Container Images with Advanced Containerfile Instructions

## Red hat universal base image (UBI)

UBI er et container image der tillader brugere at bygge containeriserede applikationer. UBI kan køre både på og udenfor redhat platforme.

Redhat får UBI fra Redhat enterprise linux men adskiller sig alligevel fra RHEL 7 base imaget, mest noterbart at det kan redistribueres under 

Red Hat Universal Base Image End User License Agreement (EULA) .

The support plan is for the Universal Base Image to follow the same  life cycle and support dates as the underlying RHEL content. When run on a subscribed RHEL or OpenShift node, it follows the same support policy as the underlying RHEL content. Red Hat maintains a Universal Base  Image for RHEL 7, which maps to RHEL 7 content, and another UBI for  RHEL 8, which maps to RHEL 8 content.

Red Hat recommends using the Universal Base Image as the base container image  for new applications. Red Hat commits to continue to support earlier  RHEL base images for the duration of the support life cycle of the RHEL  release.

The Red Hat Universal Base Image consists of:

- A set of three base images (`ubi`, `ubi-minimal`, and `ubi-init`). These mirror what is provided for building containers with RHEL 7 base images.
- A set of language runtime images (`java`, `php`, `python`, `ruby`, `nodejs`). These runtime images enable developers to start developing applications with the confidence that a Red Hat built and supported container image  provides.
- A set of associated Yum  repositories and channels that include RPM packages and updates. These  allow you to add application dependencies and rebuild container images  as needed

## UBI typer

- ubi

  A standard base image built on enterprise-grade packages from RHEL. Good for most application use cases.

- ubi-minimal

  A minimal base image built using `microdnf`, a scaled down version of the `dnf` utility. This provides the smallest container image.

- ubi-init

  This image allows you to easily run multiple services, such as web servers,  application servers, and databases, all in a single container. It allows you to use the knowledge built into systemd unit files without having  to determine how to start the service.

## Fordele ved UBI

- **Minimal size:** The Universal Base Image is a relatively minimal (approximately 90-200 MB) base container image with fast startup times.
- **Security:** Provenance is a  huge concern when using container base images. You must use a trusted  image, from a trusted source. Language runtimes, web servers, and core  libraries such as OpenSSL have an impact on security when moved into  production. The Universal Base Image receives timely security updates  from Red Hat security teams.
- **Performance:** The base images  are tested, tuned, and certified by a Red Hat internal performance  engineering team.  These are proven container images used extensively in some of the world's most compute-intensive, I/O intensive, and fault  sensitive workloads.
- **ISV, vendor certification, and partner support:** The Universal Base Image inherits the broad ecosystem of RHEL partners, ISVs, and third-party vendors supporting thousands of applications. The Universal Base Image makes it easy for these partners to build, deploy, and certify their applications and allows them to deploy the resulting  containerized application on both Red Hat platforms such as RHEL and  OpenShift, as well as non-Red Hat container platforms.
- **Build once, deploy onto many different hosts:** The Red Hat Universal Base Image can be built and deployed anywhere: on OpenShift/RHEL or any other container host (Fedora, Debian, Ubuntu, and more).

# Containerfiles

En containerfil automatiserer bygningen af container images. En containerfil er en tekstfil der indeholder et set af instruktuioner om hvordan man bygger et container image. 

Det er ligesom en docker file

**RUN**

Run commadoen eksekverer kommandoer i et nyt lag ovenpå et container image og committer resultatet.

For mange run instruktioner kan resultere i et image der er unødvendigt stort, da hver run instruktion skaber et nyt lag.

**LABEL**

Definerer image metadata såsom beskrivelser, navn, version, tag osv.

**Table 2.1. OpenShift Supported Labels**

| Label Name                     | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| `io.openshift.tags`            | This label contains a list of  comma-separated tags. Tags categorize container images into broad areas  of functionality. Tags help UI and generation tools to suggest relevant  container images during the application creation process. |
| `io.k8s.description`           | This label provides consumers of the  container image more detailed information about the service or  functionality that the image provides. |
| `io.openshift.expose-services` | This label contains a list of service ports that match the `EXPOSE` instructions in the Containerfile and provides more descriptive information about what actual service is provided to consumers. The format is `PORT[/PROTO]:NAME` where `[/PROTO]` is optional and it defaults to `tcp` if not specified. |

**WORKDIR**

Sætter working directory for en efterfølgende RUN, CMD, ENTRYPOINT, COPY eller ADD instruktioner i en containerfil.

Redhat foreslår at man bruger absolutte værdier i en workdir instruktion

**ENV**

Definerer enviornment variabler der er tilgæbngelige for containeren.

**USER**

Redhat foreslår at man ikke kører som en root user hvis man kan ungå det.

**Volume**

Laver et mount point indeni containeren der indikerer til image consumers at volumes i host maskinen eller andre containere kan bindes til dette mount point.

## Building Images with the ONBUILD Instruction

ONBUILD instruktionen registrer triggere indeni container imaget. En containerfil bruger ONBUILD til at erklære instruktioner der kun eksekveres når man bygger et child image.



For example, assume you are building a Node.js parent image that you  want all developers in your organization to use as a base when they  build applications with the following requirements:

- Enforce certain standards, such as copying the JavaScript sources to the  application folder, so that they are interpreted by the Node.js engine.

- Execute the `npm install` command, which fetches all dependencies described in the `package.json` file.

You cannot embed these requirements as instructions in the parent  Containerfile because you do not have the application source code, and  each application may have different dependencies listed in their `package.json` file.

Declare `ONBUILD` instructions in the parent Containerfile. The parent Containerfile is shown below:

```
FROM registry.access.redhat.com/rhscl/nodejs-6-rhel7
EXPOSE 3000
# Make all Node.js apps use /opt/app-root as the main folder (APP_ROOT).
RUN mkdir -p /opt/app-root/
WORKDIR /opt/app-root

# Copy the package.json to APP_ROOT
ONBUILD COPY package.json /opt/app-root

# Install the dependencies
ONBUILD RUN npm install

# Copy the app source code to APP_ROOT
ONBUILD COPY src /opt/app-root

# Start node server on port 3000
```

Assuming you built the parent container image and called it `mynodejs-base`, a child Containerfile for an application that uses the parent image appears as follows:

```
FROM mynodejs-base
RUN echo "Started Node.js server..."
```

When the build process for the child image starts, it triggers the execution of the three `ONBUILD` instructions defined in the parent image, before invoking the `RUN` instruction in the child Containerfile.

## OpenShift Considerations for the USER Instruction

By default, OpenShift runs containers using an arbitrarily assigned  userid. This approach mitigates the risk of processes running in the  container getting escalated privileges on the host machine due to  security vulnerabilities in the container engine.

#### Adapting Containerfiles for OpenShift

When you write or change a Containerfile that builds an image to run on an OpenShift cluster, you need to address the following:

- Directories and files that are read from or written to by processes in the container should be owned by the `root` group and have group read or group write permission.
- Files that are executable should have group execute permissions.
- The processes running in the container must not listen on privileged ports  (that is, ports below 1024), because they are not running as privileged  users.

Adding the following RUN instruction to your Containerfile sets the directory and file permissions to allow users in the `root` group to access them in the container:

```
RUN chgrp -R 0 directory && \
    chmod -R g=u directory
```

#### Running Containers as root Using Security Context Constraints (SCC)

In certain cases, you may not have access to Containerfiles for certain images. You may need to run the image as the `root` user. In this scenario, you need to configure OpenShift to allow containers to run as `root`.

OpenShift provides *Security Context Constraints (SCCs)*, which control the actions that a pod can perform and which resources it can access. OpenShift ships with a number of built-in SCCs. All  containers created by OpenShift use the SCC named `restricted` by default, which ignores the userid set by the container image, and assigns a random userid to containers.

To allow containers to use a fixed userid, such as 0 (the `root` user), you need to use the *anyuid* SCC. To do so, you first need to create a *service account*. A service account is the OpenShift identity for a pod. All pods from a  project run under a default service account, unless the pod, or its  deployment configuration, is configured otherwise.

If you have an  application that requires a capability not granted by the restricted  SCC, then create a new, specific service account, add it to the  appropriate SCC, and change the deployment configuration that creates  the application pods to use the new service account.

The following steps detail how to allow containers to run as the `root` user in an OpenShift project:

- Create a new service account:

  ```
  [user@host ~]$ oc create serviceaccount myserviceaccount
  ```

- Modify the deployment configuration for the application to use the new service account. Use the `oc patch` command to do this:

  ```
  [user@host ~]$ oc patch dc/demo-app --patch \
  '{"spec":{"template":{"spec":{"serviceAccountName": "myserviceaccount"}}}}'
  ```

### Note

For details on how to use the `oc patch` command, see [OpenShift admins guide to oc patch](https://access.redhat.com/articles/3319751). Run the `oc patch -h` command to display usage.

- Add the `myserviceaccount` service account to the `anyuid` SCC to run using a fixed userid in the container:

  ```
  [user@host ~]$ oc adm policy add-scc-to-user anyuid -z myserviceaccount
  ```