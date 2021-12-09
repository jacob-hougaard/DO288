## 6.6 - Guided Exercise: Customizing Deployments with Kustomize

You should be able to customize the Famous Quotes application deployment and deploy it to an RHOCP cluster.

```bash
lab multicontainer-kustomize start
```

I denne excercise skal vi opsætte 3 enviornments. Deployment, Staging og Produktion. Disse 3 enviornments har forskellige behov i forhold til performance og antal af kørende containere

Find the requirements of each environment in the following table:

| Environment | Pods | Memory | CPU Limit |
| ----------- | ---- | ------ | --------- |
| Development | 1    | 128Mi  | 250m      |
| Staging     | 2    | 256Mi  | 500m      |
| Production  | 5    | 512Mi  | 1000m     |

## Undersøg famous quotes deployment filen

```
cd ~/DO288/labs/multicontainer-kustomize
```

```
cat famous-quotes.yaml
```



## Lav folder strukturen for kustomize.

```bash
mkdir famous-kustomize
```

```bash
cd famous-kustomize
```

Lav base refrencen

```bash
mkdir base
```

```bash
cp ../famous-quotes.yaml base/deployment.yaml
```

Lav resource beskrivelses filen for base definitionen ved at lave en ny kustomization.yaml fil i base folderen for at holde base kustomize definitionen. Tilføj deployment.yaml filen som en resource i kustomization.yaml filen.

```bash
resources:
- deployment.yaml
```

## Test base definitionen

```bash
source /usr/local/etc/ocp4.config
```

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```bash
oc new-project ${RHT_OCP4_DEV_USER}-multicontainer-kustomize
```

**Tilføj kustomize base definitionen** 

Kommandoen under laver en ny deployment kaldet famousapp. For at verificere status af denne deployment, brug `oc get deployments`

```bash
oc apply -k base
```

```
oc get deployments
```

Exposer applikations servicen

```bash
oc expose service famousapp-famouschart
```

```bash
FAMOUS_URL=$(oc get route -n ${RHT_OCP4_DEV_USER}-multicontainer-kustomize  famousapp-famouschart -o jsonpath='{.spec.host}'/random)
```

Følgende curl request giver et random quote hver gang den køres

```bash
curl $FAMOUS_URL
```

## Lav deployment enviornment definitionen

```bash
mkdir -p overlays/dev
```

lav replica_limits.yaml filen i overlays/dev directoriet for at holde ændringerne i deployment definitionen

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: famousapp-famouschart
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: famouschart
          resources:
            limits:
              memory: "128Mi"
              cpu: "250m"
```

lav overlays/dev/kustomization.yaml filen og tilføj base folderen some base definition og replica_limits.yaml filen som patch

```
bases:
- ../../base
patches:
- replica_limits.yaml
```

Tilføj deployment overlays til den kørende instans af applikationen

```bash
oc apply -k overlays/dev
```

Verificer at applikationen stadigvæk kører

```bash
curl $FAMOUS_URL
```

Verificer at kustomized tilføjede memory limiten

```
oc get deployments famousapp-famouschart -o jsonpath='{.spec.template.spec.containers[0].resources.limits.memory}' 128Mi
```

## Lav staging enviornment

```bash
cp -R overlays/dev overlays/stage
```

Modificer configurationen i overlays/stage/replica_limits til at matche følgende:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: famousapp-famouschart
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: famouschart
          resources:
            limits:
              memory: "256Mi"
              cpu: "500m"
```

apply staging overlay

```bash
oc apply -k overlays/stage
```

```bash
curl $FAMOUS_URL
```

```bash
oc get deployments famousapp-famouschart -o jsonpath='{.spec.template.spec.containers[0].resources.limits.memory}'
```

## Lav produktions enviornment

Lav produktions enviornment ved at kopiere development enviornment 

```
cp -R overlays/dev overlays/prod
```

Modificer overlays/prod/replica filen

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: famousapp-famouschart
spec:
  replicas: 5
  template:
    spec:
      containers:
        - name: famouschart
          resources:
            limits:
              memory: "512Mi"
              cpu: "1000m"
```

Apply konfiguration

```
oc apply -k overlays/prod
```

Test endpoint

```
curl $FAMOUS_URL
```

Tjek at vi bruger 512mb memory

```bash
oc get deployments famousapp-famouschart \
-o jsonpath='{.spec.template.spec.containers[0].resources.limits.memory}'
```

afslut

