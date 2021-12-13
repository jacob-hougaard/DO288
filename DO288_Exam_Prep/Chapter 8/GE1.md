```
source /usr/local/etc/ocp4.config
```

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc project ${RHT_OCP4_DEV_USER}-external-service
```

```
oc status
```

```
oc get pod
```

```
oc rsh todoapp-1-wtbmr env | grep DATABASE
```

```
HOSTNAME=$(oc get route todoapp -o jsonpath='{.spec.host}')
```

```
echo ${HOSTNAME}
```

```
curl -si  http://${HOSTNAME}/todo/api/items/6
```

```
dbhost=$(echo mysql.ocp-${RHT_OCP4_WILDCARD_DOMAIN#"apps."})
```

```
echo ${dbhost}
```

```
mysqlshow -h${dbhost} -utodoapp -predhat123 todo
```

```
oc create svc externalname tododb --external-name ${dbhost}
```

```
oc get svc
```

```
oc delete project ${RHT_OCP4_DEV_USER}-external-service
```

```
lab external-service finish
```