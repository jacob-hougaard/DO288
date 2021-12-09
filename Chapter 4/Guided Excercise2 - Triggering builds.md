# Triggering builds

Formål

- Create an application that relies on an S2I builder image.
- Push a new version of the S2I builder image.
- Update metadata inside an image stream to react to the image update.
- Verify that the application is rebuilt to use the new S2I builder image.

Start lab

```
lab trigger-builds start
```

```
source /usr/local/etc/ocp4.config
```

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

Lav et nyt projekt

```
oc new-project ${RHT_OCP4_DEV_USER}-trigger-builds
```

Login i podman

```
podman login -u ${RHT_OCP4_QUAY_USER} quay.io
```

Gå til trigger-builds repoet

```
cd /home/student/DO288/labs/trigger-builds
```

Push image til quay.io

```
skopeo copy oci-archive:php-73-ubi8-original.tar.gz docker://quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8:latest
```

Quay.io defaulter til private images. Så derfor skal du tilføje en secret til builder service accounten for at få adgang til det.

Lav en fra container registriet api access token der er gemt af podman

```
oc create secret generic quay-registry --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type kubernetes.io/dockerconfigjson
```

Tilføj Quay.io registry secret til builder service kontoen.

```
oc secrets link builder quay-registry
```

Update the `php` image stream to fetch the metadata for the new container image. The external registry uses the docker-distribution package and does not notify Red Hat OpenShift about image changes.

```
oc import-image php --from quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8 --confirm
```

Create a new application build with the following parameters:

- Name: `trigger`
- Builder image: `php`
- Directory of the application: `trigger-builds`

```
oc new-app --name trigger php~http://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps --context-dir trigger-builds
```

Vent til builded afslutter

```bash
oc logs -f bc/trigger
```

Vent på at applikationen er klar og kører 

```bash
oc get pods
```

Se at image triggeren er defineret som en del af build configurationen

```bash
oc describe bc/trigger | grep Triggered
```

Opdater image streamen for at starte et nyt build

```
skopeo copy oci-archive:php-73-ubi8-newer.tar.gz docker://quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8:latest
```

Update the `php` image stream to fetch the metadata for the new container image.

```bash
oc import-image php
```

The image stream update triggered a new build. List all builds to verify that a second build started:

```bash
oc get builds
```

```
cd
```

Slet

```
skopeo delete docker://quay.io/${RHT_OCP4_QUAY_USER}/php-73-ubi8
```

