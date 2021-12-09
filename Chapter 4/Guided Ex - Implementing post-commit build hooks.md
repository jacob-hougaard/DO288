# Guided Exercise: Implementing Post-Commit Build Hooks

```bash
source /usr/local/etc/ocp4.config
```

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc project ${RHT_OCP4_DEV_USER}-post-commit
```

```
oc new-app --name hook --context-dir post-commit php:7.3~http://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps
```

```
oc logs -f bc/hook
```

```
oc get pods
```

```
cat ~/DO288/labs/post-commit/create-hook.sh
```

```
~/DO288/labs/post-commit/create-hook.sh
```

```
oc describe bc/hook | grep Post
```

```
oc start-build bc/hook -F
```

```
oc get builds
```

```
oc set env bc/hook DEVELOPER="Your Name"
```

```
oc set env bc/hook --list
```

```
firefox $(oc get route/builds-for-managers \
-o jsonpath='{.spec.host}') &
```

```
oc delete project ${RHT_OCP4_DEV_USER}-post-commit
```

```
lab post-commit finish
```