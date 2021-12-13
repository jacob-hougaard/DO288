# Guided Exercise: Activating Probes

*In this exercise, you will configure liveness and readiness probes to monitor the health of an application deployed to your Red Hat OpenShift cluster.*

You should be able to:

- Configure readiness and liveness probes for an application from the command line.
- Locate probe failure messages in the event log.

## 1 - Lav projekt og deployment

```bash
lab probes start
```

```bash
source /usr/local/etc/ocp4.config
```

```bash
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```bash
oc new-project ${RHT_OCP4_DEV_USER}-probes
```

Lav en deployment config med følgende parametre

- Name: `probes`
- Build image: `nodejs:12`
- Application directory: `probes`
- Build variables:
  - Name: `npm_config_registry`, value: `http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs`

```
oc new-app --name probes --context-dir probes --build-env npm_config_registry=http://${RHT_OCP4_NEXUS_SERVER}/repository/nodejs nodejs:12~https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps
```

Undersøg build logs og vent til den er færdig med at bygge

```bash
oc logs -f bc/probes
```

```bash
oc get pods
```

## Test applicationens ready og health endpoints

```bash
oc expose svc probes
```

```
curl -i probes-${RHT_OCP4_DEV_USER}-probes.${RHT_OCP4_WILDCARD_DOMAIN}/ready
```

```
curl -i probes-${RHT_OCP4_DEV_USER}-probes.${RHT_OCP4_WILDCARD_DOMAIN}/healthz
```

Test appens hovedside

```
curl probes-${RHT_OCP4_DEV_USER}-probes.${RHT_OCP4_WILDCARD_DOMAIN}
```

## Aktiver readiness og liveness probes for applikationen

Brug oc set kommandoen for at konfigurere liveness og readyness probes med følgende parametre.

- For the liveness probe, use the `/healthz` endpoint on the port `8080`.
- For the readiness probe, use the `/ready` endpoint on the port `8080`.
- For both probes:
  - Configure initial delay of `2` seconds.
  - Configure the timeout of `2` seconds.

```bash
oc set probe deployment probes --liveness --get-url=http://:8080/healthz --initial-delay-seconds=2 --timeout-seconds=2
```

```bash
oc set probe deployment probes --readiness --get-url=http://:8080/ready --initial-delay-seconds=2 --timeout-seconds=2
```

Verificer vores probes

```bash
oc describe deployment probes | grep -iA 1 liveness
```

```bash
oc get pods
```

Brug logs kommandoen for at se status på liveness og readiness probes.

```bash
POD=$(oc get pods -o name | grep -v build)
```

```bash
oc logs -f $POD
```

## Simuler en network failure

```
~/DO288/labs/probes/kill.sh
```

```
oc logs -f $POD
```

```
oc get pods
```

Check app loggen

```
POD=$(oc get pods -o name | grep -v build)
```

```
oc logs -f $POD
```

## Simuler en fejl med at nå applikationens database.

```bash
~/DO288/labs/probes/not-ready.sh
```

```bash
oc logs -f $POD
```

**ouptut**

```bash
ping /ready => pong [notready]
ping /healthz => pong [healthy]
...output omitted...
```

Se pods

```bash
oc get pods
```

Check at servicen ikke er tilgængelig

```bash
source /usr/local/etc/ocp4.config
```

```bash
curl -is probes-${RHT_OCP4_DEV_USER}-probes.${RHT_OCP4_WILDCARD_DOMAIN} | grep 'HTTP/1.0'
```

output

```
HTTP/1.0 503 Service Unavailable
```

## Verificer at du kan se fejlen for proberne i applikations loggen

```bash
POD=$(oc get pods -o name | grep -v build)
```

```bash
oc describe $POD
```

**output**: 

```bash
Events:
  Type     Reason     Age  From   Message
  ----     ------     ---- ----   -------
...output omitted...
  Warning  Unhealthy  ...  ...    Liveness probe failed: ... statuscode: 503
  Normal   Killing    ...  ...    Container probes failed liveness probe, will be restarted
...output omitted...
  Warning  Unhealthy  ...  ...    Readiness probe failed: ... statuscode: 503
```

## Cleanup

```bash
oc delete project ${RHT_OCP4_DEV_USER}-probes
```

```bash
lab probes finish
```

