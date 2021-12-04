# Managing an Application with the CLI

## Formål med øvelsen

- Lær at lave en applikation fra en custom template. Denne template deployer en applications pod fra PHP source kode og en database pod fra en MySQL server container image.
- Identificer root cause af en deployment error ved hjælp af Openshift CLI'en
- Fix erroren vha. openshift CLI'en

 `~/DO288/labs/build-template/php-mysql-ephemeral.json` er der en build config der ser ud som under.

```bash
{
    "kind": "Template",
    "apiVersion": ""template.openshift.io/v1",
    "metadata": {
        "name": "php-mysql-ephemeral",  
...output omitted...
    "objects": [
        {
            "apiVersion": "v1",
            "kind": "Secret",  
...output omitted...
            },
            "stringData": {
                "database-password": "${DATABASE_PASSWORD}",
                "database-user": "${DATABASE_USER}"
...output omitted...
        {
            "apiVersion": "route.openshift.io/v1",
            "kind": "Route",  
...output omitted...
            "spec": {
                "host": "${APPLICATION_DOMAIN}",
                "to": {
                    "kind": "Service",
                    "name": "${NAME}"
...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO1-1) | The template is a copy of the standard `cakephp-mysql-example` template, with resources and parameters specific to that framework  deleted. The custom template is suitable for any simple PHP application  that uses a MySQL database. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO1-2) | A secret stores database login credentials and populates environment  variables in both the application and the database pods. OpenShift `secrets` are explained later in this book. |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO1-3) | A route provides external access to the application.         |

Templaten definerer også resourcer for en PHP applikation. Det følgende fokuserer på build konfigurationen og omitter servcen og image stream ressourcerne.

```bash
...output omitted...
        {
            "apiVersion": "build.openshift.io/v1",
            "kind": "BuildConfig",  
...output omitted...
                "source": {
                    "contextDir": "${CONTEXT_DIR}",
                    "git": {
                        "ref": "${SOURCE_REPOSITORY_REF}",
                        "uri": "${SOURCE_REPOSITORY_URL}"
                    },
                    "type": "Git"
                },
                "strategy": {
                    "sourceStrategy": {
                        "from": {
                            "kind": "ImageStreamTag",  
                            "name": "php:7.3-ubi8",
...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO2-1) | A build configuration uses the S2I process to build and deploy a PHP  application from source code. The build configuration and associated  resources are the same as those that would be created by the `oc new-app` command on the Git repository. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO2-2) | A standard OpenShift image stream provides the PHP runtime builder image. |

Denne del af templaten definerer ressourcer for en MySql database. Den følgende listing fokuserer på deployment og omitter service og image stream ressourcer.

```bash
...output omitted...
        {
            "apiVersion": "apps/v1",
            "kind": "Deployment",  
...output omitted...
                        "containers": [
...output omitted...
                                "name": "mysql",
                                "ports": [
                                    {
                                        "containerPort": 3306
...output omitted...
                        "volumes": [
                            {
                                "emptyDir": {},  
                                "name": "data"
...output omitted...
                "triggers": [
...output omitted...
                            "env": {
...output omitted...
                                "image": "mysql:5.7",  
...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO3-1) | A deployment deploys a MySQL database container. The deployment and  associated resources are the same as those that would be created by the `oc new-app` command on the database image. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO3-2) | The database is not backed by persistent storage. All data would be lost if the database pod is restarted. |
| [![3](https://rol.redhat.com/rol/static/roc/Common_Content/images/3.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO3-3) | A standard OpenShift image stream provides the MySQL database image. |

Endelig definerer templaten også nogle få parametre. Nogle af disse parametre er listed underneden. Du vil bruge flere af dem.

```
...output omitted...
    "parameters": [
        {
            "name": "NAME",
            "displayName": "Name",
            "description": "The name assigned to all of the app objects defined in this template.",
...output omitted...
        {
            "name": "SOURCE_REPOSITORY_URL",  
            "displayName": "Git Repository URL",
            "description": "The URL of the repository with your application source code.",
...output omitted...
        {
            "name": "DATABASE_USER",  
            "displayName": "Database User",
...output omitted...
```

| [![1](https://rol.redhat.com/rol/static/roc/Common_Content/images/1.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO4-1) | Parameters provide application-specific configurations, such as the Git repository URL. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [![2](https://rol.redhat.com/rol/static/roc/Common_Content/images/2.svg)](https://rol.redhat.com/rol/app/#deploying-manage-cli-ge-practice-CO4-2) | The template's parameters also include database configuration and connection credentials. |

## Installering af vores custom template

Først og fremmest, source configurations filen

```bash
source /usr/local/etc/ocp4.config
```

Log derefter ind i openshift.

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

Led efter en template der bruger PHP og MySQL. Openshift leverer en template baseret på CakePHP frameworket.

```bash
oc get templates -n openshift | grep php | grep mysql
```

Denne template er dog for advanceret til det vi vil opnå, og vi bruger derfor den givne template i stedet.

**Lav et ny projekt**

```bash
oc new-project ${RHT_OCP4_DEV_USER}-common
```

```bash
oc create -f ~/DO288/labs/build-template/php-mysql-ephemeral.json
```

**Deploy applikationen ved hjælp af vores custom template**

```bash
oc new-project ${RHT_OCP4_DEV_USER}-build-template
```

Review vores template parametre for at bestemme hvilke der er påkrævet for at deploye vores quotes applikation

```bash
oc describe template php-mysql-ephemeral -n ${RHT_OCP4_DEV_USER}-common
```

resultatet ser ud som følger:

```bash
Name:		php-mysql-ephemeral
...output omitted...
Parameters:
    Name:           NAME
    Display Name:   Name
    Description:    The name assigned to all of the app objects defined in this template.
    Required:       true
    Value:          php-app
...output omitted...
```

Lig mærke til at værdierne her matcher vores YAML fil fra tidligere.

Review create-app.sh scriptet

```bash
cat ~/DO288/labs/build-template/create-app.sh
```

Kør create app scriptet

```bash
~/DO288/labs/build-template/create-app.sh
```

Følg loggen

```bash
oc logs -f bc/quotesapi
```

Vent på at applikationen er oppe og køre 

```bash
oc get pod
```

Find routen og tilgå den for at se din applikation kører

```bash
oc get route
```

## Troubleshoot forbindelse mellem applikationen og database podden

Applikationen virker ikke som forventet, men build og deployment processerne var successfulde. Fejlen er muligvis sket pga. en applikationsfejl, en manglende prerequisite eller en ukorrekt configuration.

**Verificer at applikationen forbinder til den korrekte database pod**

```
[student@workstation DO288-apps]$ oc describe svc quotesdb | grep Endpoints
Endpoints:		10.129.0.142:3306
[student@workstation ~]$ oc describe pod quotesdb-6b7ffcc649-dslpq | grep IP
IP:			10.129.0.142
```

**Verificer database pod login credentials**

```bash
oc describe pod quotesapi-7d76ff58f8-6j2gx | grep -A 4 Environment
```

**Verificer database forbindelses parametrene i applications podden**

```bash
oc describe pod quotesapi-7d76ff58f8-6j2gx | grep -A 5 Environment
```

Notice that the environment variables that provide the database login credentials match between the two pods:

- `DATABASE_NAME` is equal to the value of `MYSQL_DATABASE`.
- `DATABASE_USER` is set to the value of the `database-user` key in the `quotesapi` secret.
- `DATABASE_PASSWORD` is set to the value of the `database-user` key in the `quotesapi` secret.
- `DATABASE_SERVICE_NAME` is equal to the database service name, which is `quotesdb`.

**Verificer at applikations podden kan nå database podden**. 

```
oc rsh quotesapi-7d76ff58f8-6j2gx bash -c 'echo > /dev/tcp/$DATABASE_SERVICE_NAME/3306 && echo OK || echo FAIL'
```

Denne kommando viser at netværksforbindelse ikke er problemet.

**Gennemse applikationsloggen for at finde grunden til fejlen og fix den.**

```bash
oc logs quotesapi-7d76ff58f8-6j2gx
```

Det viser sig at det er her fejlen ligger. 

```bash
[Mon May 27 14:54:56.187516 2019] [php7:notice] [pid 52] [client 10.128.2.3:43952] SQL error: Table 'phpapp.quote' doesn't exist\n
10.128.2.3 - - [27/May/2019:14:54:51 +0000] "GET /get.php HTTP/1.1" 500 - "-" "curl/7.29.0"
```

**Kopier SQL scriptet til database podden**

```bash
oc cp ~/DO288/labs/build-template/quote.sql quotesdb-6b7ffcc649-dslpq:/tmp/quote.sql
```

**Eksekver database scriptet**

```bash
oc rsh -t quotesdb-6b7ffcc649-dslpq
```

```bash
mysql -u$MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE < /tmp/quote.sql
```

```
curl -si \http://quote-$RHT_OCP4_DEV_USER.$RHT_OCP4_WILDCARD_DOMAIN/get.php
```