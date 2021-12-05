# Building container images with advanced...

Outcomes

- Create an Apache HTTP Server container image using a Containerfile and deploy it to an OpenShift cluster.
- Create a child container image by extending the parent Apache HTTP Server image.
- Change the Containerfile for the child container image so that it runs on an OpenShift cluster with a random user id.

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```bash
oc new-project ${RHT_OCP4_DEV_USER}-container-build
```

```bash
podman build --layers=false -t do288-apache ./container-build
```

```bash
podman tag do288-apache quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```

```bash
podman login quay.io -u ${RHT_OCP4_QUAY_USER}
```

```bash
podman push quay.io/${RHT_OCP4_QUAY_USER}/do288-apache
```