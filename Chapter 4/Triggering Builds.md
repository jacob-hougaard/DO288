# Triggering builds



## Definering af build triggers

Det er muligt at definerere build triggers i Openshift for at lave nye builds ud fra bestemte events. Du kan bruge disse build triggers for at holde dine applikations containere opdateret med vilkårlige nye container images 

**Image change triggers**

Genbygger en applikations container image for at incorperere ændringer lavet af parent imaget

**Webhook triggers**

Openshift webhook triggers er HTTP API endpoints der kan starte et nyt build. Brug et webhook for at integrere nye 

Oc new-app kommandoen laver automatisk image triggers for applikationer ved brug af enten source eller docker strategier:

- For source er parent image S2I builder imaget for applikations programmeringssproget
- For docker strategien er parent imaget refereret i FROM instruktionen i applikationens Containerfile

For at se triggers associeret fmed en build konfiguration, brug `oc describe bc` kommandoen

```bash
oc describe bc/name
```

For at tilføje en change trigger til en build configuration, brug oc set triggers kommandoen.

`oc et triggers bc/name --from-image=project/image:tag`

En enkelt build configuration kan ikke indeholde flere image change triggers.

For at fjerne en image change trigger fra en build configuration, brug `oc set triggers` kommandoen med --remove option

```bash
oc set triggers bc/name --from-image=project/image:tag --remove
```

## Starting new builds with webhook triggers.

Openshift giver specialliserede webhook typer der understøtter api endpoints der er compatible med følgende VCS services

* Gitlbab
* Github
* Bitbucket

Der findes også en generisk webhook type der tager en payload defineret af openshift. Et generisk webhook kan blive brugt af en vilkårlig software til at starte et redhat openshift build. For alle webhooks er det nødvendigt at definere en secret med en key ved navn WebHookSecretKey og værdien skal invokeres når webhooket starter.

`oc new-app` kommandoen laver et generisk og et Git webhook. For at tilføje flere typer webhooks til en build configuration, brug `oc set triggers` kommandoen.

For eksempel: For at tilføje et Gitlab webhook til en build konfiguration:

```bash
oc set triggers bc/name --from-gitlab
```

If the build configuration already includes a  GitLab webhook, the previous command resets the authentication secret  embedded in the URL. You must update your GitLab projects to use the new webhook URL.

To remove an existing webhook from a build configuration, use the `oc set triggers` command with the `--remove` option. For example, to remove a GitLab webhook from a build configuration:

```bash
oc set triggers bc/name --from-gitlab --remove
```

oc set triggers kommandoen understøtter også --from-github og --from-bitbucket.

To retrieve a webhook URL and the secret, use the `oc describe` command and look for the specific type of webhook you need.