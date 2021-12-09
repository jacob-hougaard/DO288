# Managing images in an Enterprise Registry

## Reviewing container registries

- Public registries

  Registries that allow anyone to consume container images directly from the  internet without any authentication. Docker Hub, Quay.io, and the  Red Hat Registry are examples of public container registries.

- Private registries

  Registries that are available only to selected consumers and usually require  authentication. The Red Hat terms-based registry is an example of a  private container registry.

- External registries

  Registries that your organization does not control. They are usually managed by a  cloud provider or a software vendor. Quay.io is an example of an  external container registry.

- Enterprise registries

  Registry servers that your organization manages. They are usually available only to the organization's employees and contractors.

- OpenShift internal registries

  A registry server managed internally by an OpenShift cluster to store  container images. Create those images using OpenShift's build  configurations and the S2I process or a Containerfile, or import them  from other registries.

### Redhat managed registries

- Red Hat Container Registry at `registry.access.redhat.com`

  It is a public registry that hosts images for Red Hat products and requires no authentication. Note that, while this  container registry is public, most of the container images that Red Hat  provides require that the user has an active Red Hat product  subscription and that they comply with the product's End-User Agreement  (EUA). Only a subset of the images available from Red Hat's public  registry are freely redistributable. These are images based on the  Red Hat Enterprise Linux Universal Base Images (UBI).

- Red Hat terms-based registry at `registry.redhat.io`

  It is a private registry that hosts images for  Red Hat products, and requires authentication. To pull images from it,  you need to authenticate with your Red Hat Customer Portal credentials.  For shared environments, such as OpenShift or CI/CD pipelines, you can  create a service account, or authentication token, to avoid exposing  your personal credentials.

- Red Hat partner registry at `registry.connect.redhat.com`

  It is a private registry that hosts images for  third-party products from certified partners. It also needs your Red Hat Customer Portal credentials for authentication. They may be subject to  subscription or licenses at the partner's discretion.

#### The Quay.io Registry

Red Hat also manages the Quay.io container registry where anyone can register  for a free account and publish their own container images.

## Deploying Enterprise Container Registries

#### Registry Server Software

Among the available registry server software are *Red Hat Quay Enterprise*, the open source Docker-Distribution server, and products such as JFrog  and Nexus. OpenShift can deploy containers from any of these registry  servers.

Red Hat Quay Enterprise is a container image registry  with advanced features such as image security scanning, role-based  access, organization and team management, image build automation,  auditing, geo-replication, and high availability.

Red Hat Quay  Enterprise provides a web interface and a REST API. It can be deployed  as a container on premises, in selected cloud providers, and also on  Red Hat OpenShift Container Platform. It is also the server software  behind Quay.io.

If you deploy Quay  Enterprise on OpenShift, it does not replace the cluster's internal  registry. A Quay Enterprise instance running on an OpenShift cluster is, for all practical purposes, like any other OpenShift application, and  it is usually available to other OpenShift clusters and any other  container hosts in your organization.

#### Authenticating with Registries

```
[user@host ~]$ podman login quay.io
Username: developer1
Password: MyS3cret!
Login Succeeded!
```

```
[user@host ~]$ podman logout quay.io
Remove login credentials for registry.redhat.io
```

```
[user@host ~]$ podman logout --all
Remove login credentials for all registries
```

Skopeo and Buildah can also use the authentication tokens stored by  Podman, but they cannot present an interactive password prompt.

Podman requires TLS and verification of the remote certificate by default. If  your registry server is not configured to use TLS, or is configured to  use a self-signed TLS certificate or a TLS certificate signed by an  unknown CA, you can add the `--tls-verify=false` option to the `login` and `pull` subcommands.

#### Managing Container Registries with Skopeo

Red Hat supports the `skopeo` command to  manage images in a container image registry. Skopeo does not use a  container engine so it is more efficient than using the `tag`, `pull`, and `push` subcommands from Podman.

Skopeo also provides additional capabilities not found in Podman, such as  signing and deleting container images from a registry server.

The `skopeo` command takes a subcommand, options, and arguments:

```
[user@host ~]$ skopeo subcommand [options] location...
```

**Main Subcommands**

- `copy` to copy images from one location to another.
- `delete` to delete images from a registry.
- `inspect` to view metadata about an image.

**Main Options**

- `--creds username:password`

  To provide login credentials or an authentication token to the registry.

- `--[src-|dest-]tls-verify=false`

  Disables TLS certificate verification.

For authentication to private registries, Skopeo can also use the same `auth.json` file created by the `podman login` command. Alternatively, you can pass your credentials on the command line, as shown below.

```
[user@host ~]$ skopeo inspect --creds developer1:MyS3cret! \
docker://registry.redhat.io/rhscl/postgresql-96-rhel7
```

Skopeo uses URIs to represent container image locations and URI  schemas to represent container image formats and registry APIs. The  following list shows the most common URI schemas:

- `oci`

  denotes container images stored in a local, OCI-formatted folder.

- `docker`

  denotes remote container images stored in a registry server.

- `containers-storage`

  denotes container images stored in the local container engine cache.

#### Pushing and Tagging Images in a Registry Server

```
skopeo copy --dest-tls-verify=false \
containers-storage:myimage \
docker://registry.example.com/myorg/myimage
```

To copy a container image from the `/home/user/myimage` OCI-formatted folder to the insecure, public registry at `registry.example.com` under the `myorg` organization or user account:

```
skopeo copy --dest-tls-verify=false \
oci:/home/user/myimage \
docker://registry.example.com/myorg/myimage
```

When copying container images between private registries, you can either authenticate to both registries using Podman before invoking the `copy` subcommand, or use the `--src-creds` and `--dest-creds` options to specify the authentication credentials, as shown below:

```
skopeo copy --src-creds=testuser:testpassword \
--dest-creds=testuser1:testpassword \
docker://srcregistry.domain.com/org1/private \
docker://dstegistry.domain2.com/org2/private
```

Arguments to the `skopeo` command are always  complete image names. The following example is an invalid command  because it provides only the registry server name as the destination  argument:

```
skopeo copy oci:myimage \
docker://registry.example.com/
```

```
skopeo copy docker://registry.example.com/myorg/myimage:1.0 \
docker://registry.example.com/myorg/myimage:latest
```

#### Deleting Images from a Registry

```
skopeo delete docker://registry.example.com/myorg/myimage
```

### Authenticating OpenShift to Private Registries

```
oc create secret docker-registry registrycreds \
--docker-server registry.example.com \
--docker-username youruser \
--docker-password yourpassword
```

Another way of creating the secret is to use the authentication token from the `podman login` command:

```
oc create secret generic registrycreds \
--from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json \
--type kubernetes.io/dockerconfigjson
```

```
 oc secrets link default registrycreds --for pull
```

```
oc secrets link builder registrycreds
```

# Allowing Access to the OpenShift Registry

I denne sektion finder vi ud af hvordan man får adgang til et openshift internal registry fra linux container værktøjer.

### Reviewing the Internal Registry

Openshift har et internt registry der er baseret på S2I. En build configuration skaber container images, enten fra source kode ller fra en containerfile og gemmer dem i et internt registry.

Det er valgfrigt at bruge det interne registry.

Some use cases exist for accessing an OpenShift internal registry outside of the S2I process. For example:

- An organization already builds container images locally, and is not yet  ready to change its development workflow. These organizations may have a private registry with limited features and want to replace it with the  OpenShift internal registry.
- An  organization maintains multiple OpenShift clusters and needs to copy  container images from a development to a production cluster. These  organizations may have a CI/CD tool that promotes images from an  internal registry to either an external registry or another internal  registry.
- An Independent Software Vendor (ISV) creates container images for its customers and publishes them to a private registry maintained by a cloud services provider, such as  Quay.io, or to an enterprise registry that the ISV maintains.

#### The Image Registry Operator

Openshift installeren konfigurer det interne registry til at være tilgængeligt indenfra openshift clustered. Exponering af det interne registry for extern access er en simpel procedure men kræver cluster administrator privilegier.

```
oc patch config.imageregistry cluster -n openshift-image-registry \
--type merge -p '{"spec":{"defaultRoute":true}}'
```

```
oc get route -n openshift-image-registry
```

#### Authenticating to an Internal Registry

```
TOKEN=$(oc whoami -t)
```

```
podman login -u myuser -p ${TOKEN} \
default-route-openshift-image-registry.domain.example.com
```

```
skopeo inspect --creds=myuser:${TOKEN} \
docker://default-route-openshift-image-registry.domain.example.com/...
```

#### Accessing the Internal Registry as a Secure or Insecure Registry

If your OpenShift cluster is configured with a valid TLS certificate for its wildcard domain, you can use the Linux container tools to work with images inside any project you have access to.

```
skopeo inspect \
docker://default-route-openshift-image-registry.domain.example.com/myproj/myapp
```

If your OpenShift cluster uses the Certification Authority (CA) that the OpenShift installer generates by default, you need to access the  internal registry as an insecure registry:

```
skopeo inspect --tls-verify=false \
docker://default-route-openshift-image-registry.domain.example.com/myproj/myapp
```

A cluster administrator can configure the route for the internal  registry in different ways, for example using an internal CA maintained  by your organization. In this scenario, your developer workstation may  or may not be already configured to trust the TLS certificate of the  internal registry.

Your organization might also retrieve the  public certificate of its OpenShift cluster's internal CA and declare it trusted inside your organization. Your cluster administrator might set  up an alternative route, with a shorter host name, to expose the  internal registry.

These scenarios are outside the scope of this  course. Refer to Red Hat Training courses on the OpenShift  administration track, such as *Red Hat OpenShift Administration I (DO280)* and *Red Hat Security: Securing Containers and OpenShift (DO425)*, for more information about configuring TLS certificates for OpenShift and your local container engine.

### Granting Access to Images in an Internal Registry

Alle brugre med adgang til et openshift registry kan pushe og pulle images til/fra et projekt. Hvis en bruger har edit eller admin roller kan de både pushe og pulle. Men hvis de har view roller kan de kun pulle images.

OpenShift also offers a few specialized roles for when you want to  grant access only to images inside a project, and not grant access to  perform other development tasks such as building and deploying  applications inside the project. The most common of these roles are:

- `registry-viewer` and `system:image-puller`

  These roles allow users to pull and inspect images from the internal registry.

- `registry-editor` and `system:image-pusher`

  These roles allow users to push and tag images to the internal registry.

The `system:*` roles provide the minimum  capabilities required to pull and push images to the internal registry.  As stated before, OpenShift users who already have `admin` or `edit` roles in a project do not need these `system:*` roles.

```
oc policy add-role-to-user system:image-puller \
user_name -n project_name
```

# Creating Image Streams

Henter informationer omkring eksterne resourcer (container images) og hiver dem ind i clustered.

Dette er et at de største værdi forskelle imellem openshift og kubernetes.

To better visualize the relationship between image streams and image stream tags, you can explore the `openshift` project that is pre-created in all OpenShift clusters. You can see  there are a number of image streams in that project, including the `php` image stream:

```
oc get is -n openshift -o name
```

A number of tags exist for the `php` image stream, and an image stream resource exists for each:

```
oc get istag -n openshift | grep php
```

If you use the `oc describe` command on an image stream, it shows information from both the image stream and its image stream tags:

```
oc describe is php -n openshift
```

#### Describing Image Names, Tags, and IDs

An image ID uniquely identifies an immutable container image using a SHA-256 hash. Remember that you can not modify a container image. Instead, you create a new container image that has a new ID. When you push a new container image to a registry server, the server associates the existing textual name with the new image ID.

When you start a container from an image name, you download the image that  is currently associated to that image name. The actual image ID behind that name may change at any moment, and the  next container you start may have a different image ID. If the image associated with an image name has any issues, and you only  know the image name, you cannot rollback to an earlier image.

Updating a container image in an external registry does not  automatically update an image stream tag. The image stream tag keeps the reference to the last image ID it fetched. This behavior is crucial to  scaling applications because it isolates OpenShift from changes  happening at a registry server.

**example**

Suppose you deploy an application from an external registry, and after a few days of testing with a few users, you decide to scale its  deployment to enable a larger user population. In the meantime, your  vendor updates the container image on the external registry. If  OpenShift had no image stream tags, the new pods would get the new  container image, which is different from the image on the original pod.  Depending on the changes, this could cause your application to fail.  Because OpenShift stores the image ID of the original image in an image  stream tag, it can create new pods using the same image ID and avoid any incompatibility between the original and updated image.

OpenShift keeps the image ID used for the first pod and ensures that new pods use the same image ID. OpenShift ensures  that all pods use exactly the same image.

To better visualize the  relationship between an image stream, an image stream tag, an image  name, and an image ID, refer to the following `oc describe is` command, which shows the source image and current image ID for each image stream tag:

```
[user@host ~]$ oc describe is php -n openshift
Name:                   php
Namespace:              openshift
...output omitted...
7.3 (latest)
  tagged from registry.redhat.io/rhscl/php-73-rhel7:latest
...output omitted...
  * registry.redhat.io/rhscl/php-73-rhel7@sha256:22ba...09b5
...output omitted...
7.2
  tagged from registry.redhat.io/rhscl/php-72-rhel7:latest
...output omitted...
  * registry.redhat.io/rhscl/php-72-rhel7@sha256:e8d6...e615
...output omitted...
```

```
[user@host ~]$ oc describe is php -n openshift
Name:                   php
Namespace:              openshift
...output omitted...
7.3 (latest)
  tagged from registry.redhat.io/rhscl/php-73-rhel7:latest
...output omitted...
  * registry.redhat.io/rhscl/php-73-rhel7@sha256:22ba...09b5
...output omitted...
    registry.redhat.io/rhscl/php-73-rhel7@sha256:bc61...1e91
...output omitted...
7.2
  tagged from registry.redhat.io/rhscl/php-72-rhel7:latest
...output omitted...
  * registry.redhat.io/rhscl/php-72-rhel7@sha256:e8d6...e615
...output omitted...
```

In the previous example, the asterisk (*) shows which image ID is the current one for each image stream tag. It is  usually the latest one to be imported, which is the first one listed.

When an OpenShift image stream tag references a container image from an  external registry, you need to explicitly update the image stream tag to get new image IDs from the external registry. By default, OpenShift  does not monitor external registries for changes to the image ID that is associated with an image name.

You can  configure an image stream tag to check the external registry for updates on a defined schedule. By default, new image stream tags do not check  for updated images.

Build configurations  automatically update the image stream tag that they use as the output  image. This forces redeployment of application pods using that image  stream tag.

### Managing Image Streams and Tags

For at lave et image stream resource for en container image hosted i et eksternt registry kan vi bruge oc import-image kommandoen. Med --confirm og --from options. Følgendne kommando updaterer en image stream tag eller laver en if der ikke findes nogen.

```
oc import-image myimagestream[:tag] --confirm \
--from registry/myorg/myimage[:tag]
```

Tag navne fra en image stream kan være forskelligt fra et container image tag på source registry serveren.

Følgende eksempel laver et 1.0 image 