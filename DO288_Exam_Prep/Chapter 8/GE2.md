# Guided Exercise: Containerizing Nexus as a Service

You should be able to:

- Build a container image from a Dockerfile.
- Verify that a helm chart uses an image stream that uses a container image from a private registry.
- Verify that a helm chart defines resource requests and resource limits for a  Java application, and overrides environment variables that would set  heap sizes.
- Configure a helm chart to use a persistent volume claim.
- Configure a helm chart to use liveness and readiness probes.
- Deploy an instance of Nexus by using the Helm chart.

## 1 - Prerequisites

```bash
lab nexus-service start
```

```bash
cd ~/DO288-apps/nexus3
```

```bash
git checkout main
```

Inspicer dockerfilen

```bash
grep VOLUME Dockerfile
```

```bash
grep NEXUS_DATA Dockerfile
```

Dockerfilen definerer NEXUS_DATA env variablen med en værdi af /nexus-data. Dockerfilen bruger denne variable for at definere en volume hvor Nexus gemmer applikations data.



Inspicer Dockerfilen og observer at den fdefinerer en env variable der sætter javas virtuelle maskines HEAP size. Senere skal denne overskrives så openshift styre pod memory settings.

```bash
grep ENV Dockerfile
```

```bash
source /usr/local/etc/ocp4.config
```

Build the container image and push it to your personal account at Quay.io. You can either run or copy and paste the commands from the `build-nexus-image.sh` script at `~/DO288/labs/nexus-service`.

```bash
podman build -t nexus3 .
```

```bash
podman login -u ${RHT_OCP4_QUAY_USER} quay.io
```

```bash
skopeo copy containers-storage:localhost/nexus3 docker://quay.io/${RHT_OCP4_QUAY_USER}/nexus3
```

## 2 - Complete the helm chart to deploy nexus as a service.

Create a copy of the starter chart to edit, and exit the `~/DO288-apps/nexus3` folder.

```bash
cp -r ~/DO288/labs/nexus-service/nexus-chart ~/
```

```bash
cd ~
```

Inspect the `~/nexus-chart/templates/imagestream.yaml` file to verify that it uses the container image set from a configuration value.

```
grep -A1 "kind: DockerImage" ~/nexus-chart/templates/imagestream.yaml
```

Open the `~/nexus-chart/values.yaml` file with a text editor and update it to set the `imageName` value to point to the container image you built. Keep this file open in the editor.

```bash
...output omitted...
imageName: quay.io/youruser/nexus3:latest
...output omitted...
```

```
nano ~/nexus-chart/values.yaml
```

Inspect the `~/nexus-chart/templates/deployment.yaml` file to verify that it defines resource requests and resource limits for the application pod:

```
grep -B1 -A5 limits: ~/nexus-chart/templates/deployment.yaml
```

Open the `~/nexus-chart/templates/deployment.yaml` file with a text editor, and then override the `INSTALL4J_ADD_VM_PARAMS` environment variable to not include any JVM heap sizing configuration,  and to define only the Java system properties required by the  application. Keep the file open in the editor.

```
apiVersion: v1
kind: Deployment
...output omitted...
      - env:
          - name: INSTALL4J_ADD_VM_PARAMS
            value: -Djava.util.prefs.userRoot=/nexus-data/javaprefs
...output omitted...
```

```
nano ~/nexus-chart/templates/deployment.yaml
```

Ændre i nexus chart yaml filen

```bash
nano ~/nexus-chart/templates/pvc.yaml
```

## 3 Create a new project. Add a secret that allows any project user to pull the Nexus container image from Quay.io.

Login i oc

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc new-project ${RHT_OCP4_DEV_USER}-nexus-service
```

```
podman login -u ${RHT_OCP4_QUAY_USER} quay.io
```

```
oc create secret generic quayio --from-file .dockerconfigjson=${XDG_RUNTIME_DIR}/containers/auth.json --type kubernetes.io/dockerconfigjson
```

4.2 - Correct helm chart before deployment



