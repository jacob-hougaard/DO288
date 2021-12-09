# Guided Exercise: Allowing Access to the OpenShift Registry

You should be able to:

- Push an image to an internal registry using the Linux container tools.
- Create a container from an internal registry using the Linux container tools.

Verify that your OpenShift cluster's internal registry is exposed.

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc get route -n openshift-image-registry
```

```
INTERNAL_REGISTRY=$( oc get route default-route -n openshift-image-registry -o jsonpath='{.spec.host}' )
```

```
echo ${INTERNAL_REGISTRY}
```

2 Push an image to the OpenShift internal registry using your developer user account.

```
oc new-project ${RHT_OCP4_DEV_USER}-common
```

```
TOKEN=$(oc whoami -t)
```

```
ls ~/DO288/labs/expose-registry/ubi-info
```

Copy the OCI image to the classroom cluster's internal registry using Skopeo and tag it as `1.0`. Use the host name and the token retrieved in previous steps.

You can cut and paste the following `skopeo copy` command from the `push-image.sh` script in the `/home/student/DO288/labs/expose-registry` folder.

```
skopeo copy --dest-creds=${RHT_OCP4_DEV_USER}:${TOKEN} oci:/home/student/DO288/labs/expose-registry/ubi-info docker://${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
```

Verify that an image stream was created to manage the new container image. The output of the `oc get is` command was edited to show each column on a single line for readability purposes because the image repository name is expected to be too long  to fit the paper width.

```
oc get is
```

3 - Create a local container from the image in the Openshift internal registry

```
podman login -u ${RHT_OCP4_DEV_USER} -p ${TOKEN} ${INTERNAL_REGISTRY}
```

Download uni-info:1.0 fra ind i den lokale container engine.

```
podman pull ${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
```

Start en ny container fra container imaget

```
podman run --name info ${INTERNAL_REGISTRY}/${RHT_OCP4_DEV_USER}-common/ubi-info:1.0
```

Slet imaget

```
oc delete is ubi-info
```

```
oc delete project ${RHT_OCP4_DEV_USER}-common
```

```
podman rm info
```

```
lab expose-registry finish
```