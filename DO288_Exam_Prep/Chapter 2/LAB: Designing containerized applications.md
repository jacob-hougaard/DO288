**Requirements**

The application is written in  Java, using the Thorntail framework. The prebuilt, runnable JAR file  (fat JAR) containing the application and the Thorntail runtime is  provided. The application provides a simple REST API that responds to  requests based on a configuration that is injected into the container as an environment variable. Build and deploy the application to an  OpenShift cluster according to the following requirements:

- The application name for OpenShift is `elvis`. The configuration for the application should be stored in a configuration map called `appconfig`.

- Deploy the application to a project named `*`youruser`*-design-container`.

- The REST API for the application should be accessible at the URL:

  `elvis-youruser-design-container.apps.cluster.domain.example.com/api/hello`.

- The Git repository and folder that contains the application sources is:

  `https://github.com/youruser/DO288-apps/hello-java`.

- The prebuilt application JAR file is available at:

  `https://github.com/RedHatTraining/DO288-apps/releases/download/OCP-4.1-1/hello-java.jar`

Navigate to your local clone of the `DO288-apps` Git repository and create a new branch named `design-container` from the `main` branch. Briefly review the Containerfile for the application in the `/home/student/DO288-apps/hello-java/` directory.

```
git checkout main
git checkout -b desing-container
git push
```

Deploy the application in the `hello-java` folder of the `DO288-apps` Git repository, using the `design-container` branch of the application. Deploy the application to the `*`youruser`*-design-container` project in OpenShift without making any changes.

Do not forget to source the variables in the `/usr/local/etc/ocp4.config` file before logging in to the OpenShift cluster.

```
source /usr/local/etc/ocp4.config
```

```bash
oc new-project ${RHT_OCP4_DEV_USER}-design-container
```

```bash
podman build --layers=false -t do288-hello-java ./hello-java
```

```bash
podman tag do288-hello-java quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java
```

```
podman push quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java
```

```bash
oc new-app --name elvis quay.io/${RHT_OCP4_QUAY_USER}/do288-hello-java
```

