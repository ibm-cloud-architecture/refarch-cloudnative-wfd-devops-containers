###### refarch-cloudnative-wfd-appetizer

## Microservices Reference Application - What's For Dinner Toolchain (IBM Containers)

This repository contains the IBM DevOps Toolchain for managing and deploying the Java microservices making up the What's For Dinner app to an __IBM Bluemix local/dedicated environment__.

The aim of this README file is to point out differences between public and local/dedicated deployment. For general info and configuration of the What's for Dinner's IBM DevOps Toolchain, __please do read [RESILIENCY](https://github.com/ibm-cloud-architecture/refarch-cloudnative-wfd-devops-containers/tree/RESILIENCY) branch first__.

[![Create wfd Deployment Toolchain](https://new-console.ng.bluemix.net/devops/graphics/create_toolchain_button.png)](https://new-console.ng.bluemix.net/devops/setup/deploy/?repository=https%3A//github.com/ibm-cloud-architecture/refarch-cloudnative-wfd-devops-containers.git&branch=DEDICATED)

### Details

Main differences between deploying the What's for Dinner application onto an IBM Bluemix public region and a local/dedicated environment are:

1. Bluemix domain.
2. IBM Bluemix Containers Service's API url.
3. IBM Bluemix Services.

#### Bluemix domain

The Bluemix domain must change appropriately so that it is now your local/dedicated bluemix domain. This can be done during this toolchain creation process.

#### IBM Bluemix Containers Service's API

IBM Bluemix Containers Service is an IBM Bluemix service which is fully supported in IBM Bluemix public cloud but __might not__ be so in local/dedicated environments. Therefore, you must figure out what the level of support for IBM Bluemix Containers Service in your local/dedicated environment is first.

Afterwards, if IBM Bluemix Containers Service is fully supported in your environment, you must appropriately specify your IBM Bluemix Containers Service's API url during this toolchain creation. You can figure this out by executing the following command:

**`cf ic info`**

#### IBM Bluemix Services

Likewise the IBM Bluemix Containers Service, there might be other IBM Bluemix services not available in you local/dedicated environment. In this case, the IBM Bluemix CloudAMQP service used to implement the message queue component of the architecture in the IBM Bluemix public cloud is most likely to be unavailable in your local/dedicated environment. However, Compose RabbitMQ might well be available.

As a result, changes have been made to the What's for Dinner application's toolchain so that it deploys a Compose RabbitMQ service and configure the various components of the application to use this service rather than the default IBM Bluemix CloudAMQP service.

Toolchain components changed to accommodate Compose RabbitMQ are the message queue component deployed plus the configuration of the menu, menu UI and turbine microservices. Since these last mentioned microservices need different credential variables to connect to Compose RabbitMQ, a new DEDICATED branch has been created for them and the toolchain will build these microservices out of those branches as a result.
