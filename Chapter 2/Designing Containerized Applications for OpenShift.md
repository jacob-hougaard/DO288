# Designing Containerized Applications for OpenShift



## Formål:

- Select an appropriate application containerization method.
- Build a container image with advanced Dockerfile directives
- Select a method for injecting configuration data  into an application and create the necessary resources to do so.

## Valg af containerizerings metode

### Valg af build metode

**Container images**

Images fra en container platform såsom dockerhub eller quai eller dockerhub

**Dockerfiler**

Der findes en dockerfil et sted i source koden. Dette kan give dig flere muligheder:

* Customizer docker imaget til og byg nye images til at understøtte din applikation, brug denne mulighed når ændringer er små og hvis du ikke vil ligge for mange lag på dit image
* Lav en ny dockerfil og customizer den originale dockerfil (brug den som parent).

**Source-to-image (S2I)**

Et source to image builder image indeholder base operativsystem biblioteker, compilers og interpreters, runtimes og Source-to-image værktøjer.

Når en applikation bliver bygget via S2I, kombinerer openshift source koden og builder imaget for at lave en ready-to-run container image som openshift kan deploye til clusteret.

Depending on the needs of your application, you have several options to use S2I builder images:

- Red Hat provides many supported S2I builder images for various types of  applications. Red Hat recommends that you use one of these standard S2I  builder images whenever possible.
- S2I  builder images are like regular container images, but with extra  metadata, scripts, and tooling included in the image. You can use  Dockerfiles to create child images based on the parent builder images  provided by Red Hat.
- If none of the  standard Red Hat-provided S2I builder images suit your application  needs, you can build your own customized S2I builder image.

## S2I builder images

S2I er en speciel slags container image der producerer et applikations container image som et output. 

For eksempel hvis du har en PHP applikation som du vil deploye til openshift, kan du bruge et PHP builder image til at producere et applikations container image. Du opgiver git repositoriet og openshift combinerer denne source kode og et base builder image for at producere en container der kan deployes på et openshift cluster. 

Dette image indeholder en version af redhat enterprise linux, et php runtime og applikationen. Builder images er en mekanisme for at gå fra kode til kørbar container hurtigt og nemt uden dockerfiler.

## Container images

Selvom S2I er den foretrukne måde at bygge et image på i openshift vil der være tidspunkter hvor images bliver leveret til dig. oc new-app kommandoen kan der refereres til et ekstern image registry (docker.io, quay.io, private hosted). 

## Redhat container catalog.

redhat container kataloget er en samling af pre-byggede images såsom databaser. Fordelen ved at bruge dette container katalog er at der er garantier for sikkerhed og stabilitet ved disse images.

## Valg af container images for applikationer.

### Red hat software collections Library (RHSCL) 

er redhats løsning for udviklere der skal bryge de nyeste udviklingsværktøjer, som ikke følger Redhat liuxes standard release schedule. 

**Red hat openshift application runtimes (RHOAR)**

Redhats udviklingsplaform for cloud-natice og microservice applikationer. RHOAR giver en redhat og optimizeret understøttet tilgang til udvikling af microservice applikationer der er målrettet openshift som udviklingsplatformen.

RHOAR understøtter flere runtimes, sprog, framework og arkitekturer. Det giver valget og fleksibiliteten til at vælge det rigtige framework og runtimes for det rigtige job. Applikationer udviklet med RHOAR kan køre på en vilkårlig infrastruktur hvor RHOCP kan køre.

RHOAR provides:

- Access to Red Hat-built and supported binaries for selected microservices development frameworks and runtimes.
- Access to Red Hat-built and supported binaries for integration modules that  replace or enhance a framework's implementation of a microservices  pattern to use OpenShift features.
- Developer support for writing applications using selected microservice  development frameworks, runtimes, integration modules, and integration  with selected external services, such as database servers.
- Production support for deploying applications using selected microservice  development frameworks, runtimes, integration modules, and integrations  on a supported OpenShift cluster.

## Oprettelse af S2I builder images

If you want your applications to use a custom S2I builder image with  your own custom set of runtimes, scripts, frameworks, and libraries, you can build your own S2I builder image. Several options exist for  creating S2I builder images:

- Create your own S2I builder image from scratch. In scenarios where your  application cannot use the S2I builder images provided by the Container  Catalog as-is, you can build a custom S2I builder image that customizes  the build process to suit your application's needs.

  OpenShift provides the `s2i` command-line tool that helps you bootstrap the build environment for  creating custom S2I builder images. It is available in the source-to-image package from the RHSCL Yum repositories (`rhel-server-rhscl-7-rpms`).

- Fork an existing S2I builder image. Rather than starting from scratch, you  can use the Dockerfiles for the existing builder images in the Container Catalog, which are available at https://github.com/sclorg/?q=s2i, and customize them to suit your needs.

- Extend an existing S2I builder image. You can also extend an existing builder  image by creating a child image and then adding or replacing content  from the existing builder image.

# Parent og child images

Ligesom et stykke kode kan extende et andet stykke kode, ligeledes kan et child image extende et parent image. 

Dette kan bla. bruges til at 

