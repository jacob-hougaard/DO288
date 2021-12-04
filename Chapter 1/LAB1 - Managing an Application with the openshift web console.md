# Managing an application with the openshift web console (LAB)



Labbet starter med at få dig til at udfylde følgende kommando :

```bash
lab deploy-image start
```

## Wildcard domain

Wildcard domænet er det domæne som du bruger til at forbinde til dit dashboard (og openshift konsollen generelt)

det kan findes i filsystemet i /usr/local/etc/ocp4.config



Det kan også findes ved at logge ind 

```bash
oc login -u ${RHT_OCP4_DEV_USER} \
-p ${RHT_OCP4_DEV_PASSWORD} ${RHT_OCP4_MASTER_API}
```

```bash
oc get route -n openshift-console
```

Der er en mængde af ting der bliver gjort her, og det er muligvis ikke nødvendigt at nævne dem alle. Det vigtige er at du kan lave en pod fra et image under administrator i Projects/dit-project/workloads (hvis der ikke allerede findes et)

Derefter er det meget nemt at opsætte en route til dette



## Delete resources from the project

1. On the Pod Details page, click Actions → Delete Pod. In the confirmation dialog box, click Delete to delete the running pod. The web console displays the Pods page.

   Wait until the Pods page shows that a new pod that was created by the deployment configuration to replace the deleted pod.

2. On the navigation bar, click Workloads → Deployment to view the Deployment page. Click hello to view the deployment details page.

   In the upper-right corner, click Actions → Delete Deployment. In the confirmation dialog box, leave the checkbox checked and click Delete to delete the deployment.

3. On the navigation bar, click Networking → Services to see that there is still a service in the project.

   Click hello to access the Service Details page. In the upper-right corner, click Actions → Delete Service. In the confirmation dialog box, click Delete to delete the service.

4. On the navigation bar, click Networking → Routes and notice that there is still a route in the project.

   Click hello-route to access the Route Details page. In the upper-right corner, click Actions → Delete Route. In the confirmation dialog box, click Delete to delete the route.

