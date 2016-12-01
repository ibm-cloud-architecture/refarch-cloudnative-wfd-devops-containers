###### refarch-cloudnative-wfd-appetizer

## Microservices Reference Application - What's For Dinner Toolchain (IBM Containers)

This repository contains the DevOps toolchain for managing and deploying the Java microservices making up the What's For Dinner app to Bluemix in IBM Containers.

### How to use

In order to create a toolchain for the What's For Dinner Microservices Reference Application, please click below on the __Create toolchain__ button and follow the instructions.

[![Create wfd Deployment Toolchain](https://new-console.ng.bluemix.net/devops/graphics/create_toolchain_button.png)](https://new-console.ng.bluemix.net/devops/setup/deploy/?repository=https%3A//github.com/ibm-cloud-architecture/refarch-cloudnative-wfd-devops-containers.git)

1. The "What is for dinner Toolchain" creation view will open.
2. In this window, you will be asked to specify the following properties for your toolchain:
 1. Specify a name for your toolchain that is __unique__ among all toolchains on your Bluemix namespace DevOps section.
 2. Click on the __GitHub__ icon. This opens the GitHub settings. Please, give your desired name to each of the repos that will be cloned.
 3. Click on the __Delivery Pipeline__ icon. This opens the delivery pipeline settings:
   * Specify the __Bluemix domain__ where your app will be hosted *(by default: mybluemix.net)*.
    * Specify the __build branch__ you would like your delivery pipelines to build the code from *(by default: master)*.
     * Specify your __app and APIs endpoints__ which must be __unique__ within Bluemix public *(by default: "mymenu" and "menu-apis" respectively)*.
      * Specify a __unique identifier__ which will be used to make the What's For Dinner microservices and their routing __unique within Bluemix public__ *(by default: toolchain's creation timestamp)*.
3. Click the Create button to complete the toolchain creation.
4. After creating the toolchain, make sure to deploy the What's For Dinner microservices in the following order:
 1. The Eureka server, by running the Eureka IC delivery pipeline.
 2. The Config server, by running the Config Server IC delivery pipeline.
 3. All other microservices, by executing their delivery pipelines.

### Details

This toolchain contains a github clone tool, and a delivery pipeline for each of the microservices.
The github clone tool creates a cloned repository for the microservice.
Each delivery pipeline consists of three stages:

1. __Build Java Projects__. This stage has only one job which runs a gradle build of the microservice.
2. __Build Docker Image__. This stage has only one job which builds a Docker image from a Dockerfile and pushes it to your Bluemix private container image registry.
3. __Deploy Microservice__. This stage has 4 jobs:
 1. **Deploy Container**. This job deploys the docker image for the new version of the microservice.
 2. **Active Deploy - Begin**. This job creates a new Active Deploy job to upgrade the microservice to a newer version. Traffic is routed to the new version of the microservice during the rampup phase and advances the Active Deploy job to the Test phase.
 3. **Test**. This job is empty by default. It can be edited to perform any required tests.
 4. **Active Deploy - Complete**. This job advances the Active Deploy job to its rampdown phase, where the old microservice version will be removed and the Active Deploy job will be marked completed if the previous test phase succeeded. Otherwise, the upgrade will be rolled back and the Active Deploy job marked as failed.

![Common pipeline](static/imgs/common-ic-pipeline.png?raw=true)

## Considerations
### Order of deployment
All microservices depend on Eureka for service registration and discovery. This is established by creating a User Provided Service (UPS) associated to the Eureka server that will hold Eureka parameters needed by the rest of the microservices such as its url. This service is created the very first time the Eureka server is deployed and it must exist before other microservices are deployed. When the rest of the microservices are deployed, they will bind to the Eureka UPS so that they can access Eureka parameters through their VCAP Services. Unfortunately, UPS and containers do not work together as expected. This is worked around by creating a [container bridge app](https://console.ng.bluemix.net/docs/containers/container_troubleshoot.html#ts_bridge_app) between the UPS and the container. Likewise, dynamic configuration is implemented using the Config server whose location need to known by the rest of the microservices during their deployment. This is done, again, by creating another UPS which is associated to the Config Server and bound to the container bridge app.

Because of the need of the User Provided Services and the container bridge app, the Eureka and Config Server __Deploy Microservice__ delivery pipeline stage contain extra jobs and look like the following:

<img src="static/imgs/eureka.png?raw=true" hspace="250">

### Eureka & Active Deploy

Active deploy switches from an old to a new version of a microservice by switching the route from the old version to the new version of that microservice. Oftentimes this is fine, but in the case of Eureka there is a complication. Eureka keeps an in-memory database of all microservices that have registered with it. Any new version of the Eureka server will not immediately have the registrations the current version has, and until it does, its service cannot fully replace the service of the old version. This means that there will be some period of time in which Eureka's service will be degraded.

After the deployment of the new version of the Eureka Server, the rest of the microservices' container groups are updated with a new value for their VCAP services containing the new Eureka Server location. As a result, an automated IBM Container mechanism will be triggered whereby as many new instances with the new VCAP services value as old instances will be spun up and once they are up and running the old instances will get shut down.

By doing this, we have observed it to take anywhere from 120-180 seconds until the new Eureka contains the registrations of all microservices, and Eureka's service is fully restored.

### Delivery pipelines & public routes

There is a limitation on this zero downtime microservices architecture approach introduced by the Bluemix Delivery Pipeline Service. The Bluemix Delivery Pipeline Service will map a public route to the microservice container group on its initial deployment and will need such route to be kept mapped to the microservice container group in order to red/black deploy a new version of the microservice.

### Multiple What's For Dinner apps

There is a limitation on the number of What's For Dinner apps you can deploy onto the same space to one. The reason for this is that the UPS created will be called likewise so then the delivery pipelines from the different What's For Dinner apps will overwrite each other's values. Hence there is a limitation of one What's For Dinner app per Bluemix Space.
