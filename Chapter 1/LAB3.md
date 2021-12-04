## Deploying and Managing Applications on an OpenShift Cluster

- Application code is deployed from a new branch, named `source-build`.
- The project name for OpenShift is `*`youruser`*-source-build`.
- The application name for OpenShift is `greet`.

- The application should be accessible from the default route:

  `greet-youruser-source-build.apps.cluster.domain.example.com`

- The Git repository that contains the application directory sources is:

  `https://github.com/yourgithubuser/DO288-apps/nodejs-helloworld`.

- Npm modules required to build the application are available from:

  `http://nexus-common.apps.cluster.domain.example.com/repository/nodejs`

  Use the `npm_config_registry` build environment variable to pass this information to the S2I builder image for Node.js.

- You can use the `python -m json.tool *`filename.json`*` command to identify syntax errors in JSON files.

1: Navigate to your local clone of the `DO288-apps` Git repository and create a new branch named `source-build` from the main branch. Deploy the application in the `nodejs-helloworld` folder to the `*`youruser`*-source-build` project on the OpenShift cluster.

Vi starter med at lave openshift projektet szmwpi-source-build

```bash
oc new-project szmwpi-source-build
```

Derefter deployer vi vores app på openshift clusteret

```bash
oc new-app --name greet -i nodejs --code https://github.com/jacob-hougaard/DO288-apps#source-build --context-dir nodejs-helloworld -e npm_config_registry=http://nexus-common.apps.eu46.prod.nextcle.com/repository/nodejs
```

2. **Display the build logs to identify the build error and inspect the application sources to determine the root cause.**

```
oc logs greet-1-build
```

Recall that you can use the `python3 -m json.tool *`filename`*` command to validate a JSON file.

3. **Fix the error in the build tool configuration file and push the changes to the Git repository.**

Det viser sig at der en fejl på linje 12. fix den og push til git

3 Start a new build of the application and verify that the application deploys successfully.

```bash
oc build greet
```

5: Verify that the application logs show no errors, expose the application  for external access, and verify that the application returns a hello,  world message.

```bash
oc logs greet-2-build
```

Vi skal expose vores app på følgende domæne

greet-szmwpi-source-build.apps.eu46.prod.nextcle.com

```
oc expose service greet --hostname=greet-szmwpi-source-build.apps.eu46.prod.nextcle.com
```

