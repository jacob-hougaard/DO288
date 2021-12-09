# Lab: Building Applications for OpenShift

## Reqs

The provided application is written in JavaScript, using the Node.js  runtime. It is a simple application based on the Express framework. You  should build and deploy the application to a Red Hat OpenShift cluster  according to the following requirements:

- The project name is `*`youruser`*-build-app`.

- The application name is `simple`. Use the `oc-new-app.sh` script in the lab's directory to create and deploy the application.  This script contains an intentional error that you fix in a later step.  Do not modify or edit the `oc-new-app.sh` script.

- The deployed application is created from the source code in the `build-app` subdirectory of the Git repository:

  `https://github.com/youruser/DO288-apps`.

- The NPM modules required to build the application are available from the Nexus server URL at:

  `http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs`.

  Use the `npm_config_registry` environment variable to pass this information to the S2I builder image for Node.js.

- The application is accessible from the default route:

  `simple-youruser-build-app.apps.cluster.domain.example.com`.

```
oc new app ${RHCP_DEV_USER}
```

```
bash /home/student/DO288/labs/build-app/oc-new-app.sh
```

```
oc logs -f buildconfig/simple
```

