# Guided Exercise: Customizing S2I Builds

På workstation VM'en, køre rhscl/httpd24-rhel7 fra en terminal og overskriv containerens entry po

```
podman run --name test -it rhscl/httpd-24-rhel7 bash
```

Inspicer S2I scriptene pakket i builder imaget. S2I scriptene er lokeret i /usr/libexec/s2i folderen.

```
cat /usr/libexec/s2i/assemble
```

```
cat /usr/libexec/s2i/run
```

```
cat /usr/libexec/s2i/usage
```



**Kig på applicatioenens source kode og custom S2I scripts**

```
cd DO288-apps
```

```
git checkout main
```

Inspicer custom S2I filer

```
cat s2i-scripts/index.html
```

Scriptene er i /home/student/DO288-apps/s2i-scripts/.s2i/bin folderen. .s2i/bin/assemble scriptsene kopierer index.html filen fra applikationens source kode til web serverens dokumenters som root ved /opt/app-root/src. Den laver også en info.html fil med page build time og enviornment information.

```
...output omitted...
 CUSTOMIZATION STARTS HERE 

echo "---> Installing application source"
cp -Rf /tmp/src/*.html ./

DATE=date "+%b %d, %Y @ %H:%M %p"

echo "---> Creating info page"
echo "Page built on $DATE" >> ./info.html
echo "Proudly served by Apache HTTP Server version $HTTPD_VERSION" >> ./info.html

 CUSTOMIZATION ENDS HERE 
...output omitted...
```

The `.s2i/bin/run` script changes the default log level of the startup messages in the web server to `debug`:

```
# Make Apache show 'debug' level logs during startup
exec run-httpd -e debug $@
```

**Deploy applikationen til et RHOCP cluster og verificer at S2I scriptet er eksekveret.**

```
source /usr/local/etc/ocp4.config
```

```
oc login -u ${RHT_OCP4_DEV_USER} -p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```
oc new-project ${RHT_OCP4_DEV_USER}-s2i-scripts
```

Kør en ny app fra git. Tilden  (~) sikrer at bruger rhscl/httpd24-rhel7 builder imaget

```
oc new-app --name bonjour httpd:2.4~https://github.com/${RHT_OCP4_GITHUB_USER}/DO288-apps --context-dir s2i-scripts
```

Vis build logs

```
oc logs -f bc/bonjour
```

**Test the application.**

```
oc get pods
```

```
oc expose svc bonjour
```

```
oc get route
```

Kald ruten

```
curl http://bonjour-${RHT_OCP4_DEV_USER}-s2i-scripts.${RHT_OCP4_WILDCARD_DOMAIN}
```

Output:

```
Hello Class! DO288 rocks!!!
```

Vis info siden

```
curl http://bonjour-${RHT_OCP4_DEV_USER}-s2i-scripts.${RHT_OCP4_WILDCARD_DOMAIN}/info.html
```

Inspicer logs

```bash
oc logs deployment/bonjour
```

**CLEANUP**

```
cd ~
```

```
oc delete project ${RHT_OCP4_DEV_USER}-s2i-scripts
```

```
podman rm test
```