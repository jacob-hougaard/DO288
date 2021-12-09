# Injecting Configuration Data into an Application

Pga. den emutable natur af containere i et openshift cluster kan configuration management blive en udfordring. Modsat traditionelle applikationer er det ikke foreslået at couple configurationen med applikationen under opstart (When running the containerized application).

Den bedste løsning er at lave en opdeling imellem de to.



**Configuration map** = Non sensitive resources

**Secrets**= Sensitive resources

#### Features of Secrets and Configuration Maps

Notice the following with respect to secrets and configuration maps:

- They can be referenced independently of their definition.
- For security reasons, mounted volumes for these resources are backed by  temporary file storage facilities (tmpfs) and never stored on a node.
- They are scoped to a namespace.

## Creating and managing secrets and configuration maps

Secrets og configuration maps er nødt til at blive lavet inden man laver de pods der er afhængige af dem. Disse kan laves enten igennem CLI'en eller web konsollen.

brug `oc create` kommandoen for at lave secrets og configuration map resources.

For at lave et nyt configurationMap der gemmer string literals.

```bash
oc create configmap config_map_name --from-literal key1=value1 --from-literal key2=value2
```

For at lave et nyt secret der gemmer string literals.

```bash
oc create secret generic secret_name --from-literal username=user1 --from-literal password=mypa55w0rd
```

To create a new configuration map that stores the contents of a file or a directory containing a set of files:

```bash
oc create configmap config_map_name --from-file /home/demo/conf.txt
```

For at få hjælp til at lave et configMap, brug kommandoen `oc create configmap --help`

Det er også muligt at lave secrets igennem openshift web konsollen

