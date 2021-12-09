```
lab manage-builds start
```

```
[student@workstation ~]$ cd DO288-apps
[student@workstation DO288-apps]$ git checkout main
```

```
[student@workstation DO288-apps]$ git checkout -b manage-builds

[student@workstation DO288-apps]$ git push -u origin manage-builds
```

```
 source /usr/local/etc/ocp4.config
```

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc new-project ${RHT_OCP4_DEV_USER}-manage-builds
```

## Create a new application.

Create a new application from sources in Git. Use the following parameters for the build:

- Application name: `jhost`
- Branch: `manage-builds`
- Build environment variable: `MAVEN_MIRROR_URL=http://${RHT_OCP4_NEXUS_SERVER}/repository/java`
- Image stream: `redhat-openjdk18-openshift:1.8`
- Build directory: `java-serverhost`

```´
oc new-app --name jhost --build-env MAVEN_MIRROR_URL=http://${RHT_OCP4_NEXUS_SERVER}/repository/java -i redhat-openjdk18-openshift:1.8 https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps#manage-builds  --context-dir java-serverhost
```



```
oc logs -f bc/jhost
```

```
oc get pods
```

```
oc expose svc/jhost
```

```
oc get route
```

Update the application to version 2.0.

Edit file

```
... code omitted ...
String msg = "I am running on server "+host+" Version `2.0 \n";`
return Response.ok(msg).build();
... code omitted ...
```

Commit the changes to the Git server.

Commit the changes to the Git server.

```
[student@workstation DO288-apps]$ cd java-serverhost
[student@workstation java-serverhost]$ git commit -a -m 'Update the version'
...output omitted...
 1 file changed, 1 insertion(+), 1 deletion(-)
 [student@workstation java-serverhost]$ git push
 [student@workstation java-serverhost]$ cd ..
```

Start a new build with the `oc start-build` command:

```
oc start-build bc/jhost
```

The application does not contain the changes that you committed to your local Git repository. The application uses the remote Git repository URL as the source. You must push the changes to the remote Git repository before you start the OpenShift build.

```
 oc cancel-build bc/jhost
```

```
oc get builds
```

Push the updated code to the git server

```bash
git push
```

```bash
oc start-build bc/jhost
```

```bash
oc get builds
```

Observe that three builds are available. The first one was created by the `oc new-app` command, the second was canceled by you, and the third features the updated application.

```
oc logs -f build/jhost-3
```

```
oc get pods
```

Test the updated application:

Clean up and delete the project:

```
oc delete project ${RHT_OCP4_DEV_USER}-manage-builds
```