# 6.4 - Guided Exercise: Creating a Helm Chart

**Outcomes**

You should be able to create a helm chart for the Famous Quotes application and its dependencies to an RHOCP cluster.

```bash
lab multicontainer-helm start
```

## **Lav en ny helm chart**

```bash
 cd ~/DO288/labs/multicontainer-helm
```

```bash
helm create famouschart
```

```bash
cd famouschart
```

Verificer strukturen

```
tree .
```

## **Configurer applikation deployment**

Brug values.yaml filen til at configurere den resulterende deployment for applikationen. I det her tilfelde skal repository delen **ændres ** til følgende:

```
image:
  repository: quay.io/redhattraining/famous-quotes
  pullPolicy: IfNotPresent
  tag: “2.1”
```

Sørg for at containeren bruger den korrekte port for at forbinde til applikationen. 

I `templates/deployment.yaml` filen, ændre containerPort propertien

```
ports:
  - name: http
    containerPort: 8000
    protocol: TCP
```

## Tilføj database dependencien

Add the `mariadb` dependency to the application chart i Chart.yaml filen.

```
dependencies:
- name: mariadb
  version: 9.3.11
  repository: https://charts.bitnami.com/bitnami
```

Opdater afhængigheden for charten

```bash
helm dependency update
```

Setup databasen for at bruge cusom værdier for authentication og sikkerhed

```
mariadb:
  auth:
    username: quotes
    password: quotespwd
    database: quotesdb
  primary:
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
```

Kan gøres ved at catte fra mariadb.yaml filen

```
cat ../mariadb.yaml >> values.yaml
```

## Konfigurer applikationens databasetilgang ved at bruge enviornment variabler.

Som default passer deployment templaten ikke nogen enviornment variabler til den deployede applikation. Modificer templates/deployment.yaml templaten til at overgive enviornment variablerne defineret i values.yaml filen til applikationens container. Tilføj følgende efter imagePullPolicy værdien i containers sektionen-

```bash
imagePullPolicy: {{ .Values.image.pullPolicy }} // Indsæt efter den her linje
env:
  {{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
  {{- end }}
```

### Warning

When dealing with YAML files, make sure that the indentation is correct: the `imagePullPolicy` and `env:` lines must be at the same level, and the `name` and `value` entries must be at the same level, deeper than the `env:` entry. The `-` in the `- name` entry must be at the same level or deeper than the `env:` entry.

**Tilføj env variabler i slutningen af values.yaml filen**

```
env:
  - name: "QUOTES_HOSTNAME"
    value: "famousapp-mariadb"
  - name: "QUOTES_DATABASE"
    value: "quotesdb"
  - name: "QUOTES_USER"
    value: "quotes"
  - name: "QUOTES_PASSWORD"
    value: "quotespwd"
```

```
cat ../env.yaml >> values.yaml
```

## Deploy the application using the Helm chart.

```
source /usr/local/etc/ocp4.config
```

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc new-project ${RHT_OCP4_DEV_USER}-multicontainer-helm
```

Brug helm install kommandoen til at deploye appen på RHOCP clusteret. Denne kommando laver en RHOCP deployment kaldet famousapp

```
helm install famousapp .
```

Verificer status

```
oc get deployments
```

Det kan tage noget tid før mariadb'en er helt færdig

```
oc get pods
```

expose service

```
oc expose service famousapp-famouschart
```

```
FAMOUS_URL=$(oc get route -n ${RHT_OCP4_DEV_USER}-multicontainer-helm  famousapp-famouschart -o jsonpath='{.spec.host}'/random)
```

```
curl $FAMOUS_URL
```

```
cd ~
```

```
lab multicontainer-helm finish
```

