## Deploying an Application to an OpenShift Cluster

Openshift supports 2 use cases 

- When the complete application life cycle, from initial development to production, is managed using OpenShift tools
- When existing containerized applications, built outside of OpenShift, are deployed to OpenShift

The *oc new-app* command creates  the resources required to build and deploy an application to OpenShift.  Different resources are created, according to the desired use case:

### oc new-app

Bruges hvis openshift skal håndtere hele applicationens livscyclus, dvs. 

- Laver build configuration der skaber applikationens container image
- Laver en deployment resource for at håndtere deployment processen
- En image-stream der peger på et genereret image eller et eksternt registry, eller til et eksisterende image i et image registry
- En service for alle porte som applicationens container image eksposerer. Hvis appliocationens cvontainer image ikke deklerer nogen porte, så kreerer oc new-app ikke en service.

Eksempel på oc new app der bygger et container image og deployer det til openshift

```bash
oc new-app \
 https://github.com/RedHatTraining/DO288/tree/main/apps/apache-httpd
```

På denne måde vil openshifts s2i selv finde ud af hvordan et docker image skal bygges.

Du kan også refere til en eksisterende container i dit image registry

```bash
oc new-app --docker-image=registry.access.redhat.com/rhel7-mysql57
```

**Add to project** knappen i web konsollen gør det samme som *oc new app* 

Selvfølgelig er det muligt at bruge oc new-app som en one-liner som beskrevet ovenover, men det er tit bedre at specificere ting, i tabellen under er der en liste over flag der kan bruges

| Option                   | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `--as-deployment-config` | Configures the oc new-app to create a DeploymentConfig resource instead of a Deployment. |
| `--image-stream`  `-i`   | Provides the image stream to be used as either the S2I builder image for an S2I build or to deploy a container image. |
| `--strategy`             | `docker` or `pipeline` or `source`                           |
| `--code`                 | Provides the URL to a Git repository to be used as input to an S2I build. |
| `--docker-image`         | Provides the URL to a container image to be deployed.        |
| `--dry-run`              | set to `true` to show the result of the operation without performing it. |
| `--context-dir`          | Provides the path to a directory to treat as the root.       |

new-app kan også tage en definition af sproget og version via følgende

```bash
oc new-app php:7.0~http://gitserver.example.com/mygitrepo
```

```bash
oc new-app -i php:7.0 http://gitserver.example.com/mygitrepo
```

Tilden og -i har samme formål.

oc new-app kan også få adgang til udestående image-registries (så længe den har adgang)

```bash
[user@host ~]$ *oc new-app \
--code http://gitserver.example.com/mygitrepo
```

```bash
[user@host ~]$ *oc new-app \
--docker-image registry.example.com/mycontainerima
```

### strategy

Openshift kan også skelne imellem hvorvidt et repository skal bygges ud fra en dockerfil eller source koden

```bash
[user@host ~]$ oc new-app \
--strategy docker  http://gitserver.example.com/mydockerfileproject
```

```bash
[user@host ~]$ oc new-app \
--strategy source  http://gitserver.example.com/user/mygitrepo
```

--image-stream og --code kan også bruges som strategy



Billedet viser hvilke resourcer der bliver lavet af oc new-app kommandoen.

<img src="..\resources\kubernetes-oc-new-app.svg" style="zoom:33%;" />

Følgende laver en applikatiopn fra source kode ved hjælp af en en deployment config

```
oc new-app --as-deployment-config \
--name hello -i php \
--code http://gitserver.example.com/mygitrepo
```

Med --name label kan alle resourcer grouperes

```
[user@host ~]$ oc new-app \
--name test http://gitserver.example.com/mygitrepo
```

Dette gør det f.eks nemmere at slette en app og dens tilhørende resourcer

```
[user@host ~]$ oc delete all -l app=test
```

