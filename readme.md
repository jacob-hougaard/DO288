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



## Vis data

```bash
oc get pods
```

```bash
oc get all
```

```bash
oc new app
```



# Teamaer

### oc new-app

Dækket i kapitel 1

# Buzzwords

**deployment config**

**image-stream**



