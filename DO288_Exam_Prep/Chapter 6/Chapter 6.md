# Chapter 6. Deploying Multi-container Applications

## Describing OpenShift Templates

❗️Templates er ved at være deprecated❗️

#### **Beskrivelse af en template**

En template er en YAML eller JSON fil der indeholder et sæt af RHOCP ressourser. Templates definerer parametre der er brugt til at customizere resource configurationer. RHOCP procesesserer templates ved at erstatte parametre referencer med værdier og bygning af customizerede set af ressourcer.

En template er god når du vil deploye et set resourcer som en enkelt unit i stedet for at deploye dem enkeltvis. Et par eksempler kan være:

- An independent software vendor (ISV) provides a template to deploy their  product on RHOCP. The template contains configuration details about the  containers that make up the product. They also contain a deployment  configuration for the number of replicas, services and routes,  persistent storage configuration, health checks, and resource limits for computing resources such as CPU, memory, and I/O.
- Your multitier application consists of a number of separate components, such as a web server, an application server, and a database. You should  deploy these components together as a single unit on RHOCP to simplify  the process of deploying the application in a staging environment. This  allows your QA and acceptance testing teams to rapidly provision  applications for testing.

#### **Template Syntax**

Syntaxen for en RHOCP template følger den generelle syntax af en RHOCP resource men med `object` attributen i stedet for `spect` attributen. En template indeholder som regel ´parametre´ og `label` attributter såvel som annotationer og metadata

```
apiVersion: template.openshift.io/v1
kind: Template 
metadata:
  name: mytemplate
  annotations:
    description: "Description" 
objects: 
- apiVersion: v1
  kind: Pod
  metadata:
    name: myapp
  spec:
    containers:
    - env:
      - name: MYAPP_CONFIGURATION
        value: ${MYPARAMETER} 
      image: myorganization/myapplication
      name: myapp
      ports:
      - containerPort: 80
        protocol: TCP
parameters: 
- description: Myapp configuration data
  name: MYPARAMETER
  required: true
labels: 
  mylabel: myapp
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-1) | Template Resource type                      |
| ------------------------------------------------------------ | ------------------------------------------- |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-2) | Optional annotations for use by RHOCP tools |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-3) | Resource list                               |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-4) | Reference to a template parameter           |
| [![5](https://rol.redhat.com/rol/static/roc/Common_Content/images/5.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-5) | Parameter list                              |
| [![6](https://rol.redhat.com/rol/static/roc/Common_Content/images/6.svg)](https://rol.redhat.com/rol/app/#multicontainer-template-lecture-CO1-6) | Label list                                  |

resource listen indeholder som regel andre resourcer såsom build configurationer, deployment configurationer, persistent volume claims (PVC), services og routes.

En vilkårlig resource liste kan refrere hvilket som helst parametre.

Du kan lave custom labels for en template

Når Templaten definerer op til flere ressourcer er det essentielt at overveje i hvilken orden disse resourcer er defineret for at imødekomme afhængigheder mellem ressourcer.

## Definition af template parametre.

The previous example template included the definition of a required  parameter. Both optional and required parameters can provide default  values. For example:

```bash
parameters:
- description: Myapp configuration data
  name: MYPARAMETER
  value: /etc/myapp/config.ini
```

RHOCP can generate random default values for parameters. This is useful for secrets and passwords:

```bash
parameters:
- description: ACME cloud provider API key
  name: APIKEY
  generate: expression
  from:"[a-zA-Z0-9]{12}"
```

## Tilføjelse af en template til openshift

for at tilføje en template kan `oc create` kommandoen samt web konsollen bruges.

## Lav en applikation fra en template

Du kan deploye en applikation direkte fra template resource definitions filen. `oc new-app` kommandoen og `oc process` kommandoen bruger template filen som input og processer den for at tilføje parametre og lave ressourcer.

Du kan også tilføje template filen til `oc create` kommandoen for at lave template resourcer i RHOCP. Hvis templaten skal genbruges af flere udviklere er det bedre at lave resourcerne i et delt projekt, hvis den skal bruges af en enkelt deployment er det bedre at gemme den i en fil.

`oc new-app` kommandoen laver resourcer fra templaten og `oc process` kommandoen laver resource lister fra tempalten. Du er nød til at enten gemme resource filen fra `oc process`  til en fil eller passe den som input til `oc create` kommandoen for at lave resourcer fra listen.

For både oc `oc new-app` kommandoen og `oc process` kommandoen er hver parameter værdy nødt til at blive givet af en anden -p option. En anden mulighed er at begge kommandoer accepterer -l parameteren der tilføjer et label til alle resourcer der er lavet fra templaten.

**Eksempler**

Deploy en app fra en template fil

```bash
oc new-app --file mytemplate.yaml -p PARAM1=value1 -p PARAM2=value2
```

Kør oc process der giver værdier til en template og gem dem i en lokal fil

```bash
oc process -f mytemplate.yaml -p PARAM1=value1 -p PARAM2=value2 > myresourcelist.yaml
```

Filen genereret fra et tidligere eksempel vil blive givet til `oc create` kommandoen

```bash
oc create -f myresourcelist.yaml
```

Du kan kombinere de tidligere to eksempler ved brug af et pipe

```bash
oc process -f mytemplate.yaml -p PARAM1=value1 -p PARAM2=value2 | oc create -f -
```

Redhat foreslå at vi bruger `oc new-app` kommandoen i stedet for `oc process` kommadoen.

```
oc process -f mytemplate.yaml --parameters
```

# Creating a helm chart

## Helm life cycle

**Table 6.1. Helm Commands**

| Command    | Description                                |
| ---------- | ------------------------------------------ |
| dependency | Manage a chart's dependencies              |
| install    | Install a chart                            |
| list       | List releases installed                    |
| pull       | Download a chart from a repository         |
| rollback   | Roll back a release to a previous revision |
| search     | Search for a keyword in charts             |
| show       | Show information of a chart                |
| status     | Display the status of the named release    |
| uninstall  | Uninstall a release                        |
| upgrade    | Upgrade a release                          |

## Helm chart structure

En helm chart består som regel af to YAML filer og en liste med templates

Yaml filerne er 

* Chart.yaml - Holder chart definitions informationen
* Values.yaml - Holder de værdier som Helm bruger i default user-lavede templates.

Basis strukturen af en Chart.yaml fil

```
apiVersion: v2 
name: mychart 
description: A Helm chart for Kubernetes 
type: application 
version: 0.1.0 
appVersion: "1.0.0" 
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-1) | Version of the Helm API to use                 |
| ------------------------------------------------------------ | ---------------------------------------------- |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-2) | Name of the Helm Chart                         |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-3) | Description of the Helm Chart                  |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-4) | Helm Chart Type: Application or Library        |
| [![5](https://rol.redhat.com/rol/static/roc/Common_Content/images/5.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-5) | Version of the Helm Chart                      |
| [![6](https://rol.redhat.com/rol/static/roc/Common_Content/images/6.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO1-6) | Version of the application this Chart packages |

Chart.yaml filen kan holde andre vilkårlige værdier, en af de vigtigste er dependency sektionen. Dependency sektionen holder en liste af andre helm charts der tillader at helm charts virker.

Dependencies sektionen kan se ud som følger

```
dependencies: 
  - name: dependency1 
    version: 1.2.3 
    repository: https://examplerepo.com/charts 
  - name: dependency2
    version: 3.2.1
    repository: https://helmrepo.example.com/charts
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO2-1) | List of dependencies                  |
| ------------------------------------------------------------ | ------------------------------------- |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO2-2) | Name of the first required Helm Chart |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO2-3) | Chart version to depend on            |
| [![4](https://rol.redhat.com/rol/static/roc/Common_Content/images/4.svg)](https://rol.redhat.com/rol/app/#multicontainer-helm-lecture-CO2-4) | Repo holding the dependent Helm Chart |

