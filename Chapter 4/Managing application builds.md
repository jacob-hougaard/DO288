# Managing Application Builds

## Lav en build configuration

- oc new-app kommandoe
  - Laver build configurationen ud fra specificerede argumenter.
- Custom build configuration
  - Lav en custom build configuration ved brug af YAML eller JSON syntax og importer det in i Openship ved hjælp af `oc create -f` kommandoen.



## Håndtering af applikations builds gennem CLI'en

Start et nyt build manuelt. Eneste argument krævet er navnet på resourcen

```bash
oc start-build <name>
```

Afbryd et build hvis det er startet med en forkert version af applikationen. For eksempel. 

```bash
oc cancel-build <name>
```

Slet en build configuration

```bash
oc delete bc/<name>
```

Slet et build (ikke en build configuration)

```bash
oc delete build/<name>
```

Beskriv en build configuration

```bash
oc describe bc name
```

Beskriv et build

```bash
oc describe build name-1
```

Vis logs for det nyeste build. -f følger liggen indtil du terminerer med CTRL+C

```bash
oc logs -f bc/<name>
```

Vis build logs for et specific build

```bash
oc logs build/<name-1>
```



## Pruning builds

Pr default er de fem nyeste builds gemt og persisteret. Du kan begrænse antallet af builds ved brug af successfulBuildHistoryLimit attributten og failedBuildsHistoryLimit attributten.

```yaml
apiVersion: "v1"
kind: "BuildConfig"
metadata:
  name: "sample-build"
spec:
  successfulBuildsHistoryLimit: 2
  failedBuildsHistoryLimit: 2
  ...contents omitted...
```

## Log verbosity.

Det er muligt at sætte log level med følgende kommando

```bash
oc set env bc/name BUILD_LOGLEVEL="4"
```

#### Pruning Builds

Kan gøres med `oc delete` eller `oc adm prune`. Kommandoen.