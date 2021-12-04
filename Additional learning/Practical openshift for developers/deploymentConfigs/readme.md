# DeploymentConfigs

## Replicas

Fortæller hvor mange instanser af en pod der skal køres. 

## How to make a deployment config using an image name

```
oc new-app <image-tag> --as-deployment-config
```

**eksempel brugt i kurset**

```
oc new-app quay.io/practicalopenshift/hello-world --as-deployment-config
```

## How to clean up deployment configs

### One by one (the hard way)

For at rydde op i en deployment config bruger man kommandoen

```bash
oc delete deploymentConfig <image-stream-tag>
```



```bash
oc delete <type>/<tag>
```

```bash
~|⇒ oc delete svc/hello-world
service "hello-world" deleted
~|⇒ oc get svc
No resources found in jacobs-project namespace.
~|⇒ oc delete dc/hello-world
deploymentconfig.apps.openshift.io "hello-world" deleted
~|⇒ oc status
In project jacobs-project on server https://api.na46.prod.nextcle.com:6443

You have no services, deployment configs, or build configs.
Run 'oc new-app' to create an application.
```

### Using label selectors

Find et label ved at lave en describe på deployment configgen

```bash
oc describe dc/hello-world
```

Brug label sektionen til at finde det rette label, i det her tilfælde app=hello-world

Nu kan vi køre oc delete med vores label selector

```bash
oc delete all -l app=hello-world
```

## Navngiv et deployment config

```bash
oc new-app \
quay.io/practicalopenshift/hello-world \
--name demo-app \
--as-deployment-config
```

