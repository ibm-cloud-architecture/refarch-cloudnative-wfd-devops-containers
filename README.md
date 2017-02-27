###### refarch-cloudnative-wfd-appetizer

## Microservices Reference Application - What's For Dinner Toolchain (IBM Containers)

This repository contains the DevOps toolchain for managing and deploying the Java microservices making up the What's For Dinner app to Bluemix in IBM Containers.

_The What’s for Dinner application has been developed and designed to run in the **IBM Bluemix us-south public region** and, accordingly, its toolchain too. Changes may be required to the toolchain if the What’s for Dinner application is to run on a different IBM Bluemix public region or on a local/dedicated environment. For local/dedicated deployments see material in the [DEDICATED](https://github.com/ibm-cloud-architecture/refarch-cloudnative-wfd-devops-containers/tree/DEDICATED) branch_

### Before you start

Before you hit the Create Toolchain button, make sure you have:

1. A [Bluemix account](https://console.ng.bluemix.net/registration/)
2. A namespace set for your IBM Containers (more info [here](https://console.ng.bluemix.net/docs/containers/container_cli_reference_cfic.html#container_cli_reference_cfic__namespace))

### How to use

In order to create a toolchain for the What's For Dinner Microservices Reference Application, please click below on the __Create toolchain__ button and follow the instructions.

[![Create wfd Deployment Toolchain](https://new-console.ng.bluemix.net/devops/graphics/create_toolchain_button.png)](https://new-console.ng.bluemix.net/devops/setup/deploy/?repository=https%3A//github.com/ibm-cloud-architecture/refarch-cloudnative-wfd-devops-containers.git&branch=RESILIENCY)

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

This RESILIENCY branch of the What's For Dinner DevOps GitHub repository not only implements the same toolchain as the master branch but also includes the new resiliency artifacts for the What's For Dinner app in its implementation.

These new elements are the Netflix OSS component Hystrix (metrics and circuit breaker), The Netflix OSS component Turbine (metric aggregator) and the CloudAMQP service (Managed HA RabbitMQ Server) for integration of these resiliency pieces with the existing microservices making up the What's For Dinner app. For further detail on these new components as well as on this new architecture that includes the resiliency pieces for the What's For Dinner app, please read the resiliency branch main app's [readme](https://github.com/ibm-cloud-architecture/refarch-cloudnative-netflix/tree/RESILIENCY).

The CloudAMQP service is used for integrating all resiliency pieces (Menu and Menu UI microservices and Turbine microservice). Therefore, the CloudAMQP service must be created and deployed before the resiliency pieces aforementioned (pipelines are numbered so that they are executed in the right order). The CloudAMQP delivery pipeline will create a new CloudAMQP service. If there is an existing CloudAMQP service already, the CloudAMQP delivery pipeline will unbind it from the container bridge app (remember that containerised microservices can only read VCAP services from Cloud Foundry apps) and then delete it before creating the new one. Finally, it will bind the new CloudAMQP service to the container bridge app and update the VCAP services of the microservices resiliency wise involved.

After deploying Menu and Menu UI microservices, which are the only microservices implementing Hystrix metrics and the Hystrix Circuit Breaker pattern, the Netflix OSS component Turbine gets deployed, so that metrics can get aggregated for later display in the Hystrix dashboard. Finally,the Netflix OSS component Hystrix gets deployed.
