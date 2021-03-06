# Nyttige kommandoer

## Lav en ny app

```bash
oc new app
```

```bash
oc new-app --as-deployment-config \ # 
--name hello \ # name/label for container
 -i php \ # definerer programeringssproget
--code http://gitserver.example.com/mygitrepo \ #repositoriet hvor koden skal tages fra
--strategy source # vi vil gerne bruge sourcve koden
```

| Option                   | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| `--as-deployment-config` | Configures the oc new-app to create a DeploymentConfig resource instead of a Deployment. |
| `--image-stream`  `-i`   | Provides the image stream to be used as either the S2I builder image for an S2I build or to deploy a container image. |
| `--strategy`             | `docker` or `pipeline` or `source`                           |
| `--code`                 | Provides the URL to a Git repository to be used as input to an S2I build. |
| `--docker-image`         | Provides the URL to a container image to be deployed.        |
| `--dry-run`              | set to `true` to show the result of the operation without performing it. |
| `--context-dir`          | Provides the path to a directory to treat as the root.       |

En liste af alle mulige kommandoer kan findes ved at køre help kommandoen:

```
oc new app -h
```

Du kan tilføje enviornment variable ved hjælp af `-e` flaget

```
oc new app my-app -e
```

 

## Vis data

```bash
oc get pods
```

Hold øje med pods

```bash
oc get pods --watch
```

```bash
oc get all
```

```bash
oc new app
```

```bash
oc status
```

## Hent fil fra et image

```bash
oc cp <container-image>:<lokation-på-host> <destination>
```

```bash
oc cp frontend-1-zvjhb:/var/log/httpd/error_log \
/tmp/frontend-server.log
```

## Dokumentation

Se dokumentation for en given resource

```bash
oc explain <name>
```

For at dykke dybere ind i en dokumentation kan man kalde resourcen og et felt inden i dokumentationen

```bash
oc explain <name>.<field>
```

**eksempel**

```bash
oc explain pod.spec
```

## Shell ind i en kørende pod

```bash
oc rsh -t <pod-name>
```

## Slet en pod

```bash
oc delete <resource-type> <name>
```



## Secrets

Set secret

```bash
oc set env deployment/mydcname \
--from secret/mysecret
```

```bash
oc set volume deployment/mydcname --add \
-t secret -m /path/to/mount/volume \
--name myvol --secret-name mysecret
```

Se secret

```bash
oc get secret/mysecret -o json
```

Slet secret

```bash
oc delete secret/mysecret
```

**Opdater secret**

Først encode din data til base64

```bash
echo 'newpassword' | base64
```

```bash
oc edit secret/mysecret
```

Kan også gøres med patch kommandoen

```bash
oc patch secret/mysecret --patch \
'{"data":{"password":"bmV3cGFzc3dvcmQK"}}'
```

## Config maps

```bash
oc set env deployment/mydcname --from configmap/myconf
```

```bash
oc set volume deployment/mydcname --add \
-t configmap -m /path/to/mount/volume \
--name myvol --configmap-name myconf
```

**Se Configmap**

```bash
oc get configmap/myconf -o json
```

```bash
oc delete configmap/myconf
```

```bash
oc edit configmap/myconf
```

## Login i et container registry med podman

```
podman login quay.io -u <my-user>
```

```
podman logout quay.io
```

```
podman logout --all
```

## Tilføj roller til en bruger

oc policy add-role-to-user system:image-puller \
user_name -n project_name

# Teamaer

### oc new-app

Dækket i kapitel 1

# Buzzwords

**deployment config**

**image-stream**



# Short-hands

| Navn | refererer til     |
| ---- | ----------------- |
| Svc  | Service           |
| bc   | build-config      |
| dc   | Deployment-config |
| cm   | configMap         |

