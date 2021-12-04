# Managing Applications with the CLI

**Hvad sker der i denne sektion?**

- Deploy en applikation fra source kode og håndtere dets resourcer ved hjælp af kommandolinje interfacet
- Beskrive openshift roller der er krævet for at kunne administrere et project og et cluster.
- Beskrive hvordan Source to image (S2I) bestemtgører builder images for en applikation.



## Openshift cluster og projekt administrations privilegier.

| Rolle                 | Adgang | Beskrivelse                                                  |
| --------------------- | ------ | ------------------------------------------------------------ |
| Cluster administrator |        | Manage projects, add nodes, create persistent volumes, assign project  quotas, and perform other cluster-wide administration tasks. |
| Project administrator |        | Manage resources inside a project, assign resource limits, and grant  other users permission to view and manage resources inside the project. |
| Developer             |        | Manage a subset of a project's resources.resources required to build and deploy applications, such as build and  deployment configurations, persistent volume claims, services, secrets,  and routes |

## Troubleshooting Build, deployments og pods igennem CLI'en

**Status information**

Kommandoer såsom `oc status` og `oc get` giver et resumé af resourcer i et projekt. 

**Resource description**

`oc describe` kommandoen viser detaljeret information omkring en resource. Her iblandt state, konfiguration og seneste begivenheder `-o` kommandoen sammen med `oc get` vil vise hele configurationen (på low-level) samt status informationer omkring en resource. 

**Resource logs**

Logs på resources kan ses igennem `oc logs` kommandoen. Kan bruges til at se applikations specifikke errors eller build errors.

**Hvis du ikke får det resultat du ønsker**

Hvis du ikke får det ønskede resultat kan du bruge `oc cp` og `oc rsh` kommandoerne til at få direkte interaktion med en containeriseret applikation.

### Sammenligning af 2 kommandoer der kan bruges til at beskrive openshift resourcer.

`oc describe` kan følge forholdet imellem resourcer imens `oc get` kommandoen viser information omkring den efterspurgte resource. For eksempel viser `oc get -o` kommandoen imod en build configuration ikke information omkring et foregående build.

For at lave ændringer til en resource kan ´oc edit´ kommandoen bruges. Denne kommando kombinerer `oc get -o` for at hente resource beskrivelsen og åbne den med en tekst editor og derefter aplier dem med `oc apply` kommandoen.

### Forbedring af containerized applikations logs.

#### Læsning af build-logs

Build logs kan læses på 3 måder. 

**Referer til build konfigurationen**

```bash
oc logs bc/myapp
```

**Referer til et specifikt build fra en build konfiguration**

```bash
oc logs build/myapp-2
```

**Referer til build podden**

```bash
oc logs myapp-build-2
```

### Direkte adgang til en containeriseret applikation

Hvis en applikation gemmer sine logs i en dets storage (ephemeral container storage) kan kommandoerne `oc cp` og `oc rsync` bruges til at hente dem ud. Disse kommandoer kan bruges til at hente en vilkårlig fil indeni en kørende containers fil system.

**eksempel**

```bash
oc cp frontend-1-zvjhb:/var/log/httpd/error_log \
/tmp/frontend-server.log
```

`oc cp` kommandoen kopierer hele foldere som default. Hvis du ønsker en enkelt fil skal destiationen også være en enkelt fil. Det er også krævet at det underliggende container image understøtter tar kommandoen, hvis dette ikke er tilfældet vil `oc cp` kommandoen fejle.

samme metode kan bruges til at kopiere filer **til** en container. dog bør dette kun bruges til midlertidige fixes.

`oc rsh` kan også bruges til at køre en kommando i en pod (eller starte en interaktiv shell)

**Kør ps ax kommandoen indeni en pod**

```
oc rsh frontend-1-zvjhb ps ax
```

**Start en shell**

```bash
oc rsh -t frontend-1-zvjhb
```

## Byg og deploy enviornment variabler

Mange containere kræver enviornment variabler når de bliver bygget. derfor kan -e optionen gives med new-app kommandoen for at definere enviornment variabler. 

Notice that a deployment configuration stores  the environment variables for application pods, while a build  configuration stores the environment variables for builder pods. Refer  to the documentation for each builder image to find information about  its build variables and their default values.References

# Referencer

Further information is available in the *Understanding Containers, Images, and Imagestreams* chapter of the *Images* guide for Red Hat OpenShift Container Platform 4.6 at https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/images/index#understanding-images