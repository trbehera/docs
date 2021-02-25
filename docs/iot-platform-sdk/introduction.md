IOT Platform SDK
================

Security Considerations
-----------------------

The Secure Device Onboard IoT Platform SDK uses several property files for configuration. It is vital that these files are protected from malicious usage. This should include setting appropriate permissions and general system security practices.

Failure to do so can severely compromise the security of this software.

Introduction
------------

The Secure Device Onboard IoT Platform SDK provides components that enable the integration of an owner's IOT Platform (also known as a Device Management Service) into a Secure Device Onboard service.  The SDK is comprised of three components: Owner Protocol Service (OPS), Owner Companion Service (OCS), and To0Scheduler.

The To0Scheduler is responsible for executing TO0, while Owner Protocol Service (OPS) is responsible for executing TO2 for any Secure Device Onboard-enabled device. The Owner Companion Service (OCS) provides the device information and functions to the other two components, and abstracts the actual owner(s) of the devices from the said components.

Components
----------

The IOT Platform SDK is comprised of the following components:

***common:***

This component holds the common packages that are shared and used by all the other components in the IOT Platform SDK package.  This component is composed of two modules:

**rest:** Contains the bean classes that represent the message objects for REST APIs, and is used during request and response messages. Also contains some utility classes to perform some specific functions.

**protocol:** Contains the Secure Device Onboard protocol-specific beans and utility classes used across the TO0 and TO2 protocol implementations. It contains the security specific implementations, encoders and decoders for Secure Device Onboard protocol messages among others.

***to0scheduler***:

This component is responsible for the scheduling and completion of TO0 for multiple ownership vouchers. This component is comprised of three modules:

**libto0**: This module is responsible for all interactions with Rendezvous service to complete TO0 for given devices. This package provides interfaces, whose implementation must provide ways to fetch the Ownership vouchers and set the TO0 status for every GUID.

**to0serviceimpl**: This module contains the spring boot application and its configuration classes. It imports the 'libto0' component as a dependency, configures it, and provides implementations of its interfaces that are used as inputs to the TO0 operation. It also implements a REST endpoint that takes an array of device IDs (GUIDs) and the corresponding TO0 wait seconds as inputs. OCS calls this endpoint to trigger TO0 for an array of devices. After receiving the device IDs, the interface implementation interacts with OCS and does TO0 for a given list of ownership vouchers. Once built, this package contains the actual runnable war with an embedded tomcat server, which the user can run to start to0scheduler as a service.

***ocs:***

This component is responsible for managing the device-specific data and operations that enables provisioning of devices to any BMS(building management service). This component is comprised of two modules:

**libocs**: Defines the contract that needs to be implemented by vendor or BMS(building management service) to integrate their solution with OPS and To0Scheduler. The implementor of the OCS needs to import this jar and provide their implementations for the defined contract.

**fsimpl**: A sample File-system implementation of OCS. It imports the 'libocs' as a dependency and implements the defined contracts. It also defines multiple REST endpoints where each endpoint is mapped to a specific contract. Being a file-based implementation, it uses the file-system as database to read from, in addition to store the device information as well as the owner credentials.

***ops:***

This component is responsible for the actual onboarding of a device by running TO2 for a given GUID. It is comprised of five modules:

**libops:** This module contains the TO2 protocol implementation. It provides interfaces whose implementations must provide the required data for the protocol to work.

**restimpl:** This module makes use of 'libops' module to execute TO2 for multiple SDO clients. It configures the 'libops' and provides implementations of a few interfaces that define the way data is provided to TO2 protocol endpoints. It wraps the TO2 protocol implementation of the 'libops' module, by creating separate REST endpoints for managing all incoming TO2 requests. It sends REST calls to OCS to get and store device-specific information, such as ownership voucher, service-info files, TO2 session information among others, then provides these as inputs to the TO2 protocol endpoints.

**epid:** This module contains the Intel<sup>®</sup> Enhanced Privacy ID(Intel<sup>®</sup> EPID)-specific implementations that is used by the 'libops' module to handle Intel<sup>®</sup> EPID-based Secure Device Onboard clients. It contacts the Intel<sup>®</sup> EPID online services to validate Secure Device Onboard clients.

**serviceinfo:** This module contains the service info interfaces and implementations to marshal these data. The interfaces must be implemented by the application that wishes to be able to transfer service info configuration data to the Secure Device Onboard clients.

## Docker Scripts

The 'demo' directory contains the Docker\* scripts, the runnable binaries, and the configuration needed to run these binaries. A single docker-compose.yml file is used to bring up the three components.

**ocs:** Contains OCS-specific information.


|||
|---|---
Config     | The configuration properties and files needed to run the ocs.war resides in this directory |
DockerFile | The Docker* file that contains the container information where the OCS can be run. |
run-ocs    | The runnable script that starts OCS as a service by running ocs.war |

**ops:** Contains OPS-specific information.

|||
|---|---
Config     | The configuration properties and files needed to run the ops.war resides in this directory.
DockerFile | The Docker* file that contains the container information where the OPS can be run.
run-ops    | The runnable script that starts OPS as a service by running ops.war.

**To0scheduler:** Contains To0Scheduler specific information.

|||
|------|---
Config          |           The configuration properties and files needed to run the to0scheduler.war resides in this directory.
DockerFile      |         The Docker* file that contains the container information where the to0scheduler can be run.
run-to0scheduler|   The runnable script that starts to0scheduler as a service by running to0scheduler.war

Terminology
-----------

Refer to the [Secure Device Onboard Reference page](../reference.md#terminology).

Development Environment Specifications
--------------------------------------

Table 1. Development Environment Specifications

| Environment Variable | Requirement
|----------------------|------------
| Operating System     | Ubuntu\* 20.04 (64-bit) or Windows\* 10 (64-bit)
| Disk space           | A minimum of 300 MB after installation
| RAM (Minimum)        | Owner: 1GB <br>  Build: 4GB
| Java\*               | Java* Development Kit 11
| Apache\*             | Apache Maven\* 3.5.4 or later (needed to build the SDK from source)
| OpenSSL\*            | Optional; for creating owner RSA/ECDSA keys and Keystore
| Docker container     | Version 18.09. Optional; for running the demo as Docker* containers.  Refer to <https://docs.docker.com/> for information on setting up Docker.                          |
| Docker Compose tool  | Version 1.21.2. Optional; for running the demo as Docker* containers.                      |
| Entropy Tool         | Tools to generate additional [entropy](<https://en.wikipedia.org/wiki/Entropy_(computing)>) in the system. Examples: 'rngd', 'haveged'      |

Reference Documents
-------------------

Refer to the [Secure Device Onboard Reference page](../reference.md#project-documentation).

Limitations of This Release
---------------------------

- The number of concurrent TO0 sessions are limited by a configuration property. TO0 requests for any numbers of devices may be dropped if this limit is reached.

- The keystore and truststore passwords are present in the configuration files as plain text. Special care must be taken to secure the configuration files.
