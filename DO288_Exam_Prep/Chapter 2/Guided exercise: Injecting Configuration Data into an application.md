## Guided Exercise: Injecting Configuration Data into an Application

**Outcomes**

- Deploy a simple Node.js-based application that prints configuration details from environment variables and files.
- Inject configuration data into the container using configuration maps and secrets.
- Change the data in the configuration map and verify that the application picks up the changed values.

## 1 Inspect the `/home/student/DO288-apps/app-config/app.js` file.

## 2 Build og deploy applicationen

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```bash
oc new-project ${RHT_OCP4_DEV_USER}-app-config
```

```bash
oc new-app --name myapp --build-env npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs nodejs:12~https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps#app-config --context-dir app-config
```

Notice there is no space before or after the equal sign (=) after `npm_config_registry`.

Se build logs 

```
oc logs -f bc/myapp
```

## 3 - Test applikationen

```bash
oc get pods
```

```bash
oc expose svc myapp
```

```bash
oc get route
```

Besøg siden, se at det ikke virker, måske mangler vi en configuration fil?

## 4 - Lav configuration map og secret resource

```bash
oc create configmap myappconf --from-literal APP_MSG="Test Message"
```

Se at configuration mappet indeholder configurations datoen

```
oc describe cm/myappconf
```

Se configurationsfil i  /home/student/DO288-apps/app-config/myapp.sec` file:

```
username=user1
password=pass1
salt=xyz123
```

```bash
oc create secret generic myappfilesec --from-file /home/student/DO288-apps/app-config/myapp.sec
```

Verificer vores secret

```bash
oc get secret/myappfilesec -o json
```

## 5 - Inject configuration map og secret ind i application containeren

```bash
oc set env deployment/myapp --from configmap/myappconf
```

```bash
oc set volume deployment/myapp --add -t secret -m /opt/app-root/secure --name myappsec-vol --secret-name myappfilesec
```

## 6 - Verificer at applicatioenen er redeployed og bruger dataen fra configurationsmappet og secret

```
oc status
```

Se at podden er redeployed 

```
oc get pods
```

Brug rsh kommandoen for at se enviornment variablerne i containeren

```
oc rsh myapp-ddffbc7f9-ntsjq env | grep APP_MSG
```

## 7 - Ændre informationen i configmap og retest appen

```
oc edit cm/myappconf
```

```
oc delete project ${RHT_OCP4_DEV_USER}-app-config
```

