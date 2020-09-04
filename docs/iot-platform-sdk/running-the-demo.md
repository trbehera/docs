Running Demos with the IOT Platform SDK
======================================

Running the IoT Platform SDK with the PRI
------------------------------------------

Secure Device Onboard provides a [Protocol Reference Implementation](https://github.com/secure-device-onboard/pri) (PRI) for the following Secure Device Onboard components: Rendezvous and Device. Both of these components are required to perform an end-to-end execution of the Secure Device Onboard protocol.

To test integration of Secure Device Onboard with the IOT Platform SDK solution, the simulated Device and Rendezvous components of the PRI can be used.

## Pre-requisites for Running the Demo

1.  Clone the source repository of the [Protocol Reference Implementation](https://github.com/secure-device-onboard/pri) (PRI). In the rest of this section, the absolute path of this folder is referred to as <sdo-pri-root\>.

2.  Build the PRI by following the steps mentioned in its [README](https://github.com/secure-device-onboard/pri/blob/master/README.md). Alternatively, run following command:

    ```
    mvn package
    ```

    The build creates and copies the Rendezvous and Device WAR/JAR files into their respective <sdo-pri-root\>/demo directory.

3.  <sdo-pri-root\> has the following directories, which are used during the IOT Platform SDK execution.

    - **<sdo-pri-root\>/demo:** Sample scripts and configuration to run the demonstration of an onboarding process are located in the following directories:
        -   README.md
        -   rendezvous/ <br>
            ./rendezvous (Sample script to start rendezvous)
        -   device/ <br>
            ./device (Sample script to start device)

4.  Clone the source repository of the [IoT Platform SDK](https://github.com/secure-device-onboard/iot-platform-sdk). In the rest of this section, the absolute path of this folder is referred to as <sdo-iot-platform-sdk-root\>.

5.  Build IOT Platform SDK by following the steps mentioned in the [README](https://github.com/secure-device-onboard/iot-platform-sdk/blob/master/README.md). Alternatively, run the following command:

    ```
    mvn clean install
    ```

    The build creates and copies the OCS, OPS, and To0Scheduler WAR/JAR files into their respective `<sdo-iot-platform-sdk-root>/demo` directory.

6.  <sdo-iot-platform-sdk-root\> has the following directories, which are used during the IOT Platform SDK execution.

    - **<sdo-iot-platform-sdk-root\>/demo:** The IOT Platform SDK root directory containing Docker* files, configurations, and execution binaries:
        -  docker-compose.yml <br>
        -  ocs/config/ <br>
           ./run-ocs (Sample script and configuration to run the OCS)
        -  ops/config/ <br>
           ./run-ops (Sample scripts and configuration to run the OPS)
        -  to0scheduler/config/ <br>
           ./run-to0scheduler (Sample scripts and configuration to run the to0scheduler)

        The example instructions are executed under the Ubuntu\* OS version 18.04.

4.  The device must have already completed the DeviceInitialization (DI) protocol, and an Owner must have been assigned to it using Supply Chain Toolkit. The resulting Ownership voucher(OV) must be moved from SCT into the OCS.


Running the Simulation Using Docker* Compose Tool
------------------------------------------------

The demo can be run using the Docker* Compose tool for the IOT Platform SDK components. Any configuration, such as configuring proxies, properties or logs, should be done before starting the services.

## Start the Simulated Rendezvous Service

Open a new terminal window and start a virtual instance of the Rendezvous service shown as follows:

```
$ cd <sdo-pri-root>/demo/rendezvous
$ sh rendezvous
```

## Start the IOT Platform SDK Services

Open a new terminal window and start the IOT Platform SDK services:

```
$ cd <sdo-iot-platform-sdk-root>/demo/
$ docker-compose up
```
This brings up the OCS, OPS, and To0Scheduler instances inside an Ubuntu-based Docker container. During this operation, everything under the Docker directory, including the configurations and binaries are copied into the built Docker containers. Only the directory 'ocs/config/db' and the file 'to0scheduler/config/redirect.properties', are configurable once the Docker container starts, to externalize the device information and values, and the TO1 OPS (owner) redirect information. If any changes are made to any of the files, other than the directory 'ocs/config/db' and the file 'to0scheduler/config/redirect.properties', both, the container and its image needs to be deleted and re-created again.

!!! Note
    When running the demo in multiple machines, the ownership voucher must be created such that the IP address of the Rendezvous service is present, instead of 'localhost' in the Rendezvous information.

## Start the Device Simulation

Open a new terminal window and follow these steps to start the device:
```
$ cd <sdo-pri-root>/demo/device
$ sh device
```

After the device script completes successfully, there will be three new files created inside <sdo-pri-root\>/demo/device:

-   linux64.sh
-   payload.bin
-   result.txt

During the onboarding process payload.bin and linux64.sh are downloaded from the Owner over an encrypted channel. This demonstrates that files can be downloaded from the OPS (Owner) and executed on the local device during the onboarding process. Once downloaded, linux64.sh is executed which in turn operates on the binary program payload.bin and creates the file result.txt:

```
$ cat result.txt
```

should output: Device onboarded successfully.

To clean up onboarded files to rerun the demo:

```
$ rm linux64.sh
$ rm payload.bin
$ rm result.txt
```

Generating Ownership Voucher/Credential Pair
--------------------------------------------
Refer to the [Supply Chain Tools Manufacturer Enablement Guide ](../supply-chain-tools/manufacturing/manufacturer-enablement-guide.md) on steps to create Ownership Voucher/Credential pair.


Working with Multiple Owner Key-Pair(s)
----------------------------------------

The OCS instance handles multiple owner key-pair(s) using a Java* Keystore. Multiple owner key-pairs can be inserted into the keystore with different aliases. The listed key tool commands require a password to be entered. Enter the password whenever prompted.

!!! Note
    A sample keystore is provided in `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/creds/owner-keystore.p12`. This is an example implementation for demo purposes and should be updated in production deployment.

## Generating Key-Pair for Owner Attestation

### Generating ECDSA Key-Pair for Owner Attestation

Secure Device Onboard specification supports the National Institute of Standards and Technology (NIST) P-256 curve and P-384 types.

**Step 1:** Generate the private key.

Generate the NIST-256 key by running the following command:

`$ openssl ecparam -genkey -name secp256r1 -out eckey.pem`

Alternatively, generate the NIST-384 key by running the following command:

`$ openssl ecparam -genkey -name secp384r1 -out eckey.pem`

**Step 2:** Generate a self-signed certificate.

Generate the self-signed certificate for the previously generated NIST-256 key as:

`$ openssl req -x509 -sha256 -nodes -days 3650 -key eckey.pem -out eccert.crt`

Alternatively, generate the self-signed certificate for the previously generated NIST-384 key as:

`$ openssl req -x509 -sha384 -nodes -days 3650 -key eckey.pem -out eccert.crt`

**Step 3:** Convert the key to public key cryptography standards (PKCS\#8) format (optional):

`$ openssl pkcs8 -topk8 -nocrypt -in eckey.pem -out eckey.key`

**Step 4:** Create a certificate signing request to send for generating a certificate chain (optional):

`$ openssl x509 -x509toreq -in eccert.crt -out CSR.csr -signkey eckey.key`

### Generating RSA Key-Pair for Owner Attestation

Secure Device Onboard specification supports the RSA2048

**Step 1:** Generate the private key.

Generate the NIST-256 key by running the following command:

`$ openssl genrsa -out rsakey.pem 2048`

**Step 2:** Generate a self-signed certificate.

Generate the self-signed certificate for the previously generated NIST-256 key as:

`$ openssl req -x509 -key rsakey.pem -days 365 -out rsacert.pem`

### Inserting an Existing Owner's Certificate and Private Key into a Keystore

Assuming that there is already an existing owner certificate named 'owner-certificate-1.pem' and the corresponding private key 'owner-private-key.pem', follow these steps to insert them as 'PrivateKeyEntry' into the keystore 'owner-keystore.p12':

**Step 1:** Convert the certificate and private key into 'PKCS12' format:

`$ openssl pkcs12 -export -in owner-certificate-1.pem -inkey
owner-private-key.pem -name owner_123 -out owner_pkcs.p12`

**Step 2:** Import the above generated PKCS12 file into the existing owner keystore file 'owner-keystore.p12' located under `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/creds` under the alias 'owner_123'. If the keystore file 'owner-keystore.p12' is not present, the keystore file is created afresh with the same name:

`$ keytool -importkeystore -destkeystore path/to/owner-keystore.p12
-srckeystore owner_pkcs.p12 -srcstoretype PKCS12 -alias owner_123`

!!! Note
    The password entered in Step 1 to generate the owner_pkcs.p12 must be the same as that of owner-keystore.p12, that is, the password of the newly created keystore must match the existing keystore where it will be imported to. This is because, in the current OCS implementation, the password of the owner-keystore.p12 is used to access the imported keys from the Step-1. As a result, every keystore being imported will have the same password as that of the destination keystore.

## Exporting an Existing Owner's Certificate from Keystore

Assuming that there is an existing owner's certificate and private key stored in the keystore as a PrivateKeyEntry under the alias 'owner_123', run the following command to extract the owner's certificate into <owner_certificate.pem\>:

**Step 1:** Get the list of key-pairs, along with their respective aliases, from the keystore:

`$ keytool -list -v -keystore /path/to/owner-keystore.p12`

**Step 2:** Export the certificate from the keystore using any of the aliases present in the keystore**:**

`$ keytool -exportcert -alias owner_123 -file
<owner_certificate.pem> -rfc -keystore /path/to/owner-keystore.p12`

## Removing an Existing Owner's Key-Pair from Keystore

Assuming that there is an existing owner's certificate and private key stored in the keystore as a PrivateKeyEntry under the alias 'owner_123', run the following command to remove the key-pair corresponding to the alias:

`$ keytool -delete -alias owner_123 -keystore owner-keystore.p12`

IOT Platform SDK and PRI Component Configuration
---------------------------------------------------------------

When the IOT Platform SDK Demo is running, log messages are displayed on the terminal window. Some log messages may be truncated for better readability, but sufficient information is provided to compare against the logs generated during a simulation.

## Configuring Proxies

Update the proxy information in _JAVA_OPTIONS as

`_JAVA_OPTIONS=-Dhttp.proxyHost=http_proxy_host -Dhttp.proxyPort=http_proxy_port -Dhttps.proxyHost=https_proxy_host -Dhttps.proxyPort=https_proxy_port`

where

-   http\_proxy\_host: Represents the http proxy hostname. Typically, it is an IP address or domain name in the proxy URL.

-   http\_proxy\_port: Represents the http proxy port. Typically, it is the port number in the proxy URL.

-   https\_proxy\_host: Represents the https proxy hostname. Typically, it is an IP address or domain name in the proxy URL.

-   https\_proxy\_port: Represents the https proxy port. Typically, it is the port number in the proxy URL.

Specify combination the hostname and port information together for either http, https, or both. For example, if the http proxy is 'http://myproxy.com:900', then the following updates will be made to the properties:

-   http\_proxy\_host: myproxy.com

-   http\_proxy\_port: 900

If no proxy needs to be specified, do not add these properties to your _JAVA_OPTIONS.

### Configuring Logs

Both the PRI and IOT Platform SDK utilizes Spring logback for logging. Each demo service has logback-spring.xml in its directory that configures the logging to be appended to the console by default. Log configuration is located in the following directories:

`<sdo-pri-root>/demo/rendezvous/logback-spring.xml`

`<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/logback-spring.xml`

`<sdo-iot-platform-sdk-root>/demo/ocs/config/logback-spring.xml`

`<sdo-iot-platform-sdk-root>/demo/ops/config/logback-spring.xml`

`<sdo-pri-root>/demo/device/logback-spring.xml`

The logback-spring.xml can be modified to disable logging or use file only logging as shown below:

Sample logback-spring.xml for logging to a file as well as on a console:

```
<?xml version=\"1.0\" encoding=\"UTF-8\"?\>
<configuration\>
  <appender name=\"FILE\" class=\"ch.qos.logback.core.FileAppender\"\>
    <file\>log-owner.txt\</file\>
    <append\>false\</append\>
    <encoder\>
      <pattern\>%d{dd-MM-yyyy HH:mm:ss.SSS} %highlight(%-5level) %magenta(\[%thread\]) %logger{36}.%M - %msg%n\</pattern\>
    </encoder\>
  </appender\>
  <appender name=\"STDOUT\"
    class=\"ch.qos.logback.core.ConsoleAppender\"\>
    <encoder\>
      <pattern\>%d{dd-MM-yyyy HH:mm:ss.SSS} %highlight(%-5level) %magenta(\[%thread\]) %logger{36}.%M - %msg%n\</pattern\>
    </encoder\>
  </appender\>
  <root level=\"info\"\>
    <appender-ref ref=\"STDOUT\" /\>
    <appender-ref ref=\"FILE\" /\>
  </root\>
</configuration\>
```
The following message is logged if the logback-spring.xml configuration file is used.

For Rendezvous Service:

`org.sdo.cri.rendezvous.RendezvousApp.logStarting - Starting RendezvousApp`

For Owner Companion Service:

`o.s.i.ocs.fsimpl.fs.FsApplication.logStarting - Starting FsApplication`

For Owner Protocol Service:

`o.s.i.ops.opsimpl.OpsApplication.logStarting - Starting OpsApplication`

For To0Scheduler Service:

`o.s.i.t.t.To0ServiceApplication.logStarting - Starting To0ServiceApplication`

For Device Simulation:

`org.sdo.cri.device.DeviceApp.logStarting - Starting DeviceApp`

## Configuring the Properties

Each Docker\* service of IOT Platform SDK has its own configuration file with extension '.env'. When each service starts, the properties stored in the '.env' files will be set as environment variables within the container. The runnable scripts, namely, run-ops, run-ocs, and run-to0scheduler, that are responsible for starting the individual services within the docker container, explicitly maps the set the environment variables to the respective property of the application and passes them as Java* system variables. The properties for the PRI components are configurable using the respective 'application.properties' file. For this simulation and during development, the '.env' files  and the 'application.properties' files are writable. However, in a production environment, change the permissions of the these files to read-only for added security.

The description for the configuration settings can be found in the
properties associated with each service.

**PRI Rendezvous Service Properties:**

`<sdo-pri-root>/demo/rendezvous/application.properties`

**PRI Device Service Properties:**

`<sdo-pri-root>/demo/device/application.properties`

**to0scheduler Docker Service Properties:**

`<sdo-iot-platform-sdk-root>/demo/to0scheduler/to0scheduler.env`

referenced by

`<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/run-to0scheduler`

**OPS Docker Service Properties:**

`<sdo-iot-platform-sdk-root>/demo/ops/ops.env`

referenced by

`<sdo-iot-platform-sdk-root>/demo/ops/config/run-ops`

**OCS Docker Service Properties:**

`<sdo-iot-platform-sdk-root>/demo/ocs/ocs.env`

referenced by

`<sdo-iot-platform-sdk-root>/demo/ocs/config/run-ocs`

For information on all the properties for PRI components, refer to the comments in the specific properties files, or the README.

For information on all the properties for IOT Platform SDK components, refer to the comments in the specific .env files.

!!! Note
    In rest of the guide, the original name of the properties specified in the runnable scripts: run-ocs, run-ops and run-to0scheduler, will be referenced, instead of the names present in the .env files. The said mapping, that is specified explicitly, generally looks like: 'PROPERTY_NAME' at .env maps to 'property.name' at the application. For example, the property reference in the guide would be 'property.name=property-value', which means that 'PROPERTY_NAME=property-value' from .env is explicitly mapped to 'property.name=${PROPERTY_NAME}' at runnable script.

## Setting up Resources for OCS

The resources used in the OCS implementation can be found in:

`<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1`

The current OCS implementation supports multiple owner key-pairs with the use of PKCS12 keystore that contains each owner's certificate and private key as a PrivateKeyEntry. The owner keystore file named 'owner-keystore.p12' is located under:

`<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/creds`

Before the onboarding of the device, the certificate and private key of the corresponding owner's public key that the OV is assigned to, must be imported into the owner keystore.

The device-specific information for every device can be found in:

`<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices`

The above directory contains several subdirectories, each representing a device, and named with the device's GUID. Each subdirectory is structured as:

-   **voucher.json**: Contains the contents of the device's ownership voucher file.

-   **state.json**: Contains the device state in the following form:

```
{
  "to2Error"     : "TO2 error message of the form {"ec":"","em":"","emsg":""}",
  "to2Timestamp" : Timestamp at which TO2 was completed,
  "to0Error"     : "TO0 error message of the form {"ec":"","em":"","emsg":""}",
  "to0Timestamp" : "Timestamp at which last TO0 was successful",
  "to2State"     : "State at which TO2 is. Can be one of to2begin/to2end/to2error",
  "to0Ws"        : "Integer value representing the number of seconds after which the last TO0 expires ",
  "g3"           : "New GUID of the device"

}
```

!!! Note
    To re-trigger TO0 scheduling for a device, delete the device's state.json file.

-   **svi.json**: An array of objects, where each object provides information about the resource (in our case, a file) that will be read/fetched to get service-info. Depending on the implementation of the OCS, the resource can be anything. For example, a database implementation of OCS may have one or more rows representing a resource.

    The 'valueId' represents a resource on the OCS, which is used by the OPS to fetch resources one-by-one in subsequent requests.

    Depending on the message field (msg), the 'valueId' in each object is representative of:

    *write* -- The ID of the resource (script, binaries, and others) that is sent to the device. In this implementation, the name of the service-info file.

    *filedesc* -- The ID of the resource that contains the name to be given to the file once it is transferred. In this implementation, the name of the file whose value is the name of the file by which it is saved at the device.

    *exec* -- The ID of the resource that contains the command that will be executed at the device. In this implementation, the name of the file whose value is the command to be executed at the device.

    The array must be ordered such that the 'filedesc' and 'write' objects  are one after the other pair-wise, followed by the 'exec' commands.

```
{
  "module"   : "Module name",
  "msg"      : "Service-info type" # One of filedesc, exec, write,
  "valueLen" : "Length"            # Length of the file to be transferred,
  "valueId"  : "ID"                # An opaque-id representing a resource (in this case, file) that contains the actual service-info value,
  "enc"      : "Encoding type"     # Encoding to be used. One of either base64, or, ascii
}
```
-   **psi.json**: Represents the pre-service info as key-value pairs that get pushed to the device. It is of the form:

```
{

  "module" : "Module name",
  "msg"    : "Any key",
  "value"  : "Any value"
}
```
-   **dvi.json**: Represents the device service-info. It is of the same form as psi.json.

    The service-info values as listed by svi.json that gets sent to device during onboarding, are found in <sdo-iot-platform-sdk-root\>/demo/ocs/config/db/v1/values

The demo onboarding process demonstrates download of a script file and a binary file. Once the files are downloaded, the script validates the checksum of the binary file and creates result.txt to store the result.

## Setting up ServiceInfo transfer
The ServiceInfo communication involves the transfer of files from the Owner to the Device.

These files can be a binary, an executable script or even a normal text file. The files that are to be transferred to the device is stored in the following path  `<iot-platform-sdk>/demo/ocs/config/db/v1/values`.

While transferring, the files are encoded in base64 format. The ServiceInfo configuration file  **_svi.json_** is available in the following path `<iot-platform-sdk>/demo/ocs/config/db/v1/devices/<DEVICE-GUID>`

The structure of **_svi.json_** can be modified to customize the ServiceInfo transfer process. For example, the following JavaScript\* Object Notation (JSON) represents that a file **'package.sh'** will be sent to the
device and will be renamed to the value stored in the file **'package_name'**, say 'linux64.sh'. Similarly, the file **'payload.bin'** will be downloaded to the device and will be renamed to the value stored in the file **'payload_name'**, say 'payload.bin'. Finally, the value of the file **'binsh-linux64'**, **'/bin/sh linux64.sh'**, will be executed as command in the device.

```json
[
  {
    "module"   : "sdo_sys",
    "msg"      : "filedesc",
    "valueLen" : -1,
    "valueId"  : "payload_name",
    "enc"      : "base64"
  },
  {
    "module"   : "sdo_sys",
    "msg"      : "write",
    "valueLen" : -1,
    "valueId"  : "payload.bin",
    "enc"      : "base64"
  },
  {
    "module"   : "sdo_sys",
    "msg"      : "filedesc",
    "valueLen" : -1,
    "valueId"  : "package_name",
    "enc"      : "base64"
  },
  {
    "module"   : "sdo_sys",
    "msg"      : "write",
    "valueLen" : -1,
    "valueId"  : "package.sh",
    "enc"      : "base64"
  },
  {
    "module"   : "sdo_sys",
    "msg"      : "exec",
    "valueLen" : -1,
    "valueId"  : "binsh-linux64",
    "enc"      : "base64"
  }
]
```

!!! Note
     The ordering of message blocks are important for **Client-SDK** / **Client-SDK TPM** device. The `filedesc` block must always be followed by `write` block and if any executable script is transfered then the `exec` block must be immediately followed after both `filedesc` and `write` blocks. Additionally, the 'filedesc' and 'write' blocks immediately preceding the 'exec' block must contain the executable script to be run. For devices except Client-SDK and Client-SDK TPM, the above order is not mandatory, we can first define `filedesc` and `write` blocks for both scripts and then define `exec` blocks later.

For the scenario of transferring multiple executables to the Device, then the following order of blocks should be maintained.
```json
[ {
      "module"   : "sdo_sys",
      "msg"      : "filedesc",
      "valueLen" : -1,
      "valueId"  : "package1_name",
      "enc"      : "base64"
    },
    {
      "module"   : "sdo_sys",
      "msg"      : "write",
      "valueLen" : -1,
      "valueId"  : "script_one.sh",
      "enc"      : "base64"
    },
    {
      "module"   : "sdo_sys",
      "msg"      : "exec",
      "valueLen" : -1,
      "valueId"  : "binsh-script1",
      "enc"      : "base64"
    },
    {
      "module"   : "sdo_sys",
      "msg"      : "filedesc",
      "valueLen" : -1,
      "valueId"  : "package2_name",
      "enc"      : "base64"
    },
    {
      "module"   : "sdo_sys",
      "msg"      : "write",
      "valueLen" : -1,
      "valueId"  : "script_two.sh",
      "enc"      : "base64"
    },
    {
      "module"   : "sdo_sys",
      "msg"      : "exec",
      "valueLen" : -1,
      "valueId"  : "binsh-script2",
      "enc"      : "base64"
    }
]
```
!!! Note
     Using a single `exec` block you cannot execute multiple scripts on the Device.

To avoid different configurations for different devices, a unified structure is followed for svi.json file which is described above and works for all devices.

!!! Note
     When running the services as Docker*, only `<sdo-iot-platform-sdk-root>/demo/ocs/config/db` and `<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/redirect.properties`, are configurable.

Given an OwnershipVoucher file with a unique deviceId/GUID, the following needs to be done to add it as a device in the directory `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices`:

1. Create a folder named deviceId/GUID.

2. Rename the Owner voucher file to 'voucher.json'. Move this file to the newly created directory.

3. Create the psi.json and svi.json files under the same directory. A sample of both files are present at the sample device directory '1fae14fb-deca-405a-abdd-b25391b9d932'.

The following is a sample Python\* script that, given an Ownership voucher file as an argument, extracts the GUID and creates the previously mentioned device directory structure. This sample script operates under the assumption that both the script and voucher files are placed at `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices`, and that this is the current working directory:

The svi.json and psi.json files are copied from the sample device  `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices/1fae14fb-deca-405a-abdd-b25391b9d932`, to the new device directory. The input voucher file is moved to the new device directory and is renamed as voucher.json.

```
import sys
import os
import os.path
import json
import base64
import codecs
import uuid
import shutil

# read the Owner Voucher (Ownership Voucher) from the /devices directory
voucherFile=open(sys.argv[1], 'r')
voucher=json.load(voucherFile)
voucherFile.close()

# read 'g' field
encodedGuid=voucher['oh']['g']

# base64 decode and convert to hex to generate the guid
guid = codecs.encode(base64.b64decode(encodedGuid), 'hex')

# format as per UUID standards
guid = uuid.UUID(guid)
print "The GUID of the input owner voucher is - " + str(guid)

# create directory with the 'guid' as its name
try:
    os.mkdir(str(guid))
except OSError:
    print ("Failed to create directory %s" % guid)
else:
    print ("Successfully created the device directory for GUID %s " % guid)

# source directory
copyFromPath = os.path.basename("1fae14fb-deca-405a-abdd-b25391b9d932")

# destination directory
copyToPath = os.path.basename(str(guid))

if os.path.isdir(copyToPath):

    # move the owner voucher represented by sys.argv[1] and rename it to voucher.json
    shutil.move(sys.argv[1], os.path.join(copyToPath, 'voucher.json'))

    print "Generated voucher.json at " + copyToPath + " with contents of input voucher " + sys.argv[1]

    # copy svi.json
    shutil.copyfile(os.path.join(copyFromPath, 'svi.json'), os.path.join(copyToPath, 'svi.json'))

    print "Copied default svi.json from device directory " + copyFromPath + " to directory " + copyToPath

    # copy psi.json
    shutil.copyfile(os.path.join(copyFromPath, 'psi.json'), os.path.join(copyToPath, 'psi.json'))

    print "Copied default psi.json from device directory " + copyFromPath + " to directory " + copyToPath
```

 For example, suppose the name of the script is add-device.py and the  name of the owner voucher file to be added as a device is device.json having GUID `fad58be5-c7ba-417c-b1bd-eb9703ea8016`. Then, the script execution would look like below.

At the end of the script execution, a device directory named `fad58be5-c7ba-417c-b1bd-eb9703ea8016` will be created at `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices` with files `voucher.json`, `svi.json`, and `psi.json` as its contents.

```
$ cd <sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/devices
$ cp path/to/device.json .
$ python add-device.py device.json
$ The GUID of the input owner voucher is - fad58be5-c7ba-417c-b1bd-eb9703ea8016

Successfully created the device directory for GUID fad58be5-c7ba-417c-b1bd-eb9703ea8016
Generated voucher.json at fad58be5-c7ba-417c-b1bd-eb9703ea8016 with contents of input voucher device.json
Copied default svi.json from device directory 1fae14fb-deca-405a-abdd-b25391b9d932 to directory fad58be5-c7ba-417c-b1bd-eb9703ea8016
Copied default psi.json from device directory 1fae14fb-deca-405a-abdd-b25391b9d932 to directory fad58be5-c7ba-417c-b1bd-eb9703ea8016
```
## Running the Owner Companion Service with HTTPS

The provided reference implementation of Owner Companion Service, uses file-system as database.

It sends REST call to To0Scheduler repeatedly at an interval (in seconds) specified by the property to0.scheduler.interval, at the URL defined by the property to0.rest.api.

By default, the server starts by using HTTPS. The keystore and truststore information can be configured, but the Mutual Transport Layer Security (TLS) authentication settings must be left untouched:

```
server.ssl.key-store-type=PKCS12 (Key-store type, Configurable)
server.ssl.trust-store-type=PKCS12 (Trust-store type, Configurable)
server.ssl.key-store=keystore.p12 (Path to keystore file, Configurable, Must be replaced for production deployment)
server.ssl.key-store-password=<password> (Keystore password, Configurable, Must be replaced for production deployment)
server.ssl.trust-store=truststore (Path to truststore file, Configurable, Must be replaced for production deployment)
server.ssl.trust-store-password=<password> (Truststore password, Configurable, Must be replaced for production deployment)
server.ssl.client-auth=need (Force Mutual TLS. Do not change)
server.ssl.ciphers=TLS_AES_256_GCM_SHA384,TLS_AES_128_GCM_SHA256 (Cipher suites to be used during TLS. Do not change)
server.ssl.enabled-protocols=TLSv1.3 (TLS version. Do not change.)
```

When starting the OCS, you can see the port(s) that the HTTPS server is listening to in the logs.
```
o.s.b.w.e.tomcat.TomcatWebServer.start - Tomcat started on port(s): 9009 (https) with context path ''
o.s.i.ocs.fsimpl.fs.FsApplication.logStarted - Started FsApplication in 7.792 seconds (JVM running for 9.125)
```
## Running the To0Scheduler with HTTPS for T00

The To0Scheduler service schedules the list of received device GUIDs from OCS, for TO0. During TO0 process, the To0Scheduler will provide the DNS, IP address, and port that the Secure Device Onboard-enabled devices must use for TO2. The To0Scheduler sends the Rendezvous server, the redirect URI that will be used by the device to send TO2 messages. This URI information is passed to the device during the TO1 process. When the device completes TO1, it uses the TO2 redirect URI to communicate with the Owner.

The TO2 redirect information, namely, DNS, IP address, and port information of the Owner Protocol Service, is stored separately in the URI specified by the property:
```
org.sdo.to0.ownersign.to1d.bo=./redirect.properties (Path to file containing redirect information, Configurable)
```
By default, the redirect information is stored in the file in the file redirect.properties, that contains the following properties:

```
dns=localhost
ip=127.0.0.1
port=8042
```
Please see the file `<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/redirect.properties` for more information on these properties and their default values.

As the TO0 scheduler runs, it will log the TO0 request and responses. On successful TO0, you see a Wait Second response, such as, ws:7200. This refers to the number of seconds the TO0 registration will be known to the Rendezvous Server.

```
o.s.i.t.to0library.To0Scheduler.setDeviceForTo0 - Register OP: 1fae14fb-deca-405a-abdd-b25391b9d932
o.s.i.t.to0library.To0ClientSession.run - <200,{"ws":7200}
o.s.i.t.t.To0SchedulerEventsImpl.onSuccess - TO0 done for the device having uuid 1fae14fb-deca-405a-abdd-b25391b9d932
```

The To0Scheduler communicates with the OCS at the URL defined by the property rest.api.server.

Similar to OCS, the server is started using HTTPS, by default. The keystore and truststore information can be configured, but the Mutual TLS authentication settings must be left untouched. The properties are the same as that of OCS.

When starting the to0scheduler service, you can see the port(s) that the HTTPS server is listening to in the logs.

```
o.s.i.t.t.To0ServiceApplication.logStarting - Starting To0ServiceApplication
o.s.b.w.e.tomcat.TomcatWebServer.start - Tomcat started on port(s): 8049 (https) with context path ''
o.s.i.t.t.To0ServiceApplication.logStarted - Started To0ServiceApplication in 9.159 seconds (JVM running for 10.693)
```
!!! NOTE
    By default, To0Scheduler is configured to verify Rendezvous service's incoming server certificate during TLS handshake for all outgoing HTTPS requests to Rendezvous service. To disable this certificate verification for demo purposes, set the application property 'org.sdo.to0.tls.test-mode' (`ORG_SDO_TO0_TLS_TEST_MODE` in `<sdo-iot-platform-sdk-root>/demo/to0scheduler/to0scheduler.env`) to 'false'.

## Starting the Owner Protocol Service with HTTP(S) for TO2 Process

The Owner Protocol server starts a HTTP server by default to listen to incoming messages from the device for TO2, by using the configuration provided in the ops.env file. To start the service using HTTPS, refer to [Enabling Transport Layer Security (TLS) during TO2](#enabling-transport-layer-security-tls-during-to2).

It communicates with the OCS at the URL defined by the rest.api.server property, using the keystore, and trustore as specified.

```
client.ssl.key-store-type=PKCS12 (Key-store type, Configurable)
client.ssl.trust-store-type=PKCS12 (Trust-store type, Configurable)
client.ssl.key-store=keystore.p12 (Path to keystore file, Configurable, Must be replaced for production deployment)
client.ssl.key-store-password=<password> (Keystore password, Configurable, Must be replaced for production deployment)
client.ssl.trust-store=truststore (Path to truststore file, Configurable, Must be replaced for production deployment)
client.ssl.trust-store-password=<password> (Truststore password, Configurable, Must be replaced for production deployment)
```

The server.port property specifies the port at which the OPS listens to for TO2 protocol connections coming from devices enabled as a Secure Device Onboard client.

When starting the owner, you can see the port(s) that the HTTP server is listening to in the logs.

The Owner Protocol Service logs the following messages when the TO2 communication starts.

```
o.s.i.o.to2library.Message40Handler.onPost -
```
When the device sends Device Service-Info message, it is logged as shown below:

```
o.s.i.o.to2library.Message46Handler.onPost -
```
Once the TO2 is completed, the following messages are logged:

```
o.s.i.o.to2library.Message50Handler.onPost -
o.s.i.o.opsimpl.OpsOwnerEventHandler.call - TO2 complete for device with guid
```
In addition, the file result.txt is created on the device containing following text:

Device onboarded successfully.

Enabling Transport Layer Security (TLS) during TO2
--------------------------------------------------

To run TO2 in TLS mode, follow the steps listed below.

**Step 1:** Add the following properties to

`<sdo-iot-platform-sdk-root>/demo/ops/ops.env`
```
SECURITY_REQUIRE_SSL=true
SERVER_SSL_KEY_STORE_TYPE=PKCS12
SERVER_SSL_KEY_STORE=ops-keystore.p12
SERVER_SSL_KEY_STORE_PASSWORD=<password>
SERVER_SSL_CIPHERS=TLS_AES_256_GCM_SHA384,TLS_AES_128_GCM_SHA256
SERVER_SSL_ENABLED_PROTOCOLS=TLSv1.3
```
Then, reference the properties in `<sdo-iot-platform-sdk-root>/demo/ops/config/run-ops`

```
SERVER_TLS_PROPERTIES="-Dsecurity.require-ssl=${SECURITY_REQUIRE_SSL}"
SERVER_TLS_PROPERTIES+=" -Dserver.ssl.key-store-type=${SERVER_SSL_KEY_STORE_TYPE}"
SERVER_TLS_PROPERTIES+=" -Dserver.ssl.key-store=${SERVER_SSL_KEY_STORE}"
SERVER_TLS_PROPERTIES+=" -Dserver.ssl.key-store-password=${SERVER_SSL_KEY_STORE_PASSWORD}"
SERVER_TLS_PROPERTIES+=" -Dserver.ssl.ciphers=${SERVER_SSL_CIPHERS}"
SERVER_TLS_PROPERTIES+=" -Dserver.ssl.enabled-protocols=${SERVER_SSL_ENABLED_PROTOCOLS}"

Add ${SERVER_TLS_PROPERTIES}, keeping the other properties intact, into the exec command as:
exec java ${SERVER_TLS_PROPERTIES} -jar -Djava.library.path=$base_dir/sdo $bin_path
```
The following properties 'server.ssl.\*' pertaining to the keystore and truststore management, must always match with the corresponding 'client.ssl.\*' properties specified for the application.

```
server.ssl.key-store-type matches client.ssl.key-store-type
server.ssl.key-store matches client.ssl.key-store
server.ssl.key-store-password matches client.ssl.key-store-password
```
**Step 2 (Optional):** A new self-signed certificate and private key can be generated by generating keystore. Create the keystore by specifying the DNS and IP address of the host machine as the subject alternative name (SAN) entries. For example, if the owner is running on a machine with DNS as owner-protocol-service.com and IP address as 10.20.30.40, then the keystore can be created by running following command.

```
$ keytool -genkeypair -keystore ops-keystore.p12 -storetype PKCS12 -storepass 123456 -alias tomcat -keyalg RSA -keysize 2048 -validity 99999 -dname "CN=localhost, OU=Development, O=Company, L=Hillsboro, ST=Oregon, C=US" -ext "san=dns:www.owner-protocol-service.com,ip:10.20.30.40"
```

**Step 3 (Optional, dependent on Step 2):** Copy the generated keystore, keystore.p12 to `<sdo-iot-platform-sdk-root>/demo/ops/config`.

**Step 4:** Export the certificate from any keystore file, say ops-keystore.p12, using following command.

```
$ openssl pkcs12 -in ops-keystore.p12 -clcerts -nokeys -out tlscert.pem
```
**Step 5:** Navigate to the $JAVA\_HOME/jre/lib/security/ folder and import the generated certificate into the Java\* device's truststore, so that the device can trust the Owner Protocol Service.

```
$ keytool -import -alias tomcat -keystore cacerts -file <PATH_TO_CERT>/tlscert.pem
```
Alternatively, to import the existing certificate into a separate truststore, run the following:

```
$ keytool -import -file <PATH_TO_CERT>/tlscert.pem -alias <ALIAS_NAME> -keystore <PATH_TO_TRUSTSTORE_FILE>
```
With these configurations in place, the OPS will enable TLS during TO2 communication.

!!! Note
    Sample keystore and truststore files are provided in `<sdo-iot-platform-sdk-root>/demo/ocs/config/`, `<sdo-iot-platform-sdk-root>/demo/ops/config/`, and `<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/`. This is an example implementation for demo purposes and should be updated in production deployment.

Running the IOT Platform SDK Components on Separate Machines
------------------------------------------------------------

To run each IOT Platform SDK component in its own separate machine, the following updates must be made:

-   Create OV/OC pair such that the DNS/IP of the rendezvous server is mentioned in these files as part of the rendezvous information. Refer to [Generating Ownership Voucher/Credential Pair](#generating-ownership-vouchercredential-pair) for steps to create such OV/OC pairs.

-   Update the following properties at each component:

 At OCS `<sdo-iot-platform-sdk-root>/demo/ocs/ocs.env`:

```
to0.rest.api=https://<To0Scheduler-machine-IP-or-DNS>:<port>/v1/to0/devices

```
 At OPS
 `<sdo-iot-platform-sdk-root>/demo/ops/ops.env`:

```
rest.api.server=https://<OCS-machine-IP-or-DNS>:<port>/
```
 At To0Scheduler
 `<sdo-iot-platform-sdk-root>/demo/to0scheduler/to0scheduler.env`:

```
rest.api.server=https://<OCS-machine-IP-or-DNS>:<port>/
```

 `<sdo-iot-platform-sdk-root>/demo/to0scheduler/config/redirect.properties`:

```
dns=<OPS-DNS-name>
ip=<OPS-machine-IP>
port=<OPS-port>
```
-   The keystore and truststore files of each of the components: OCS, OPS, and To0Scheduler needs to be updated. Both the keystore and truststore files must contain the certificates whose Common Name (CN) or the Subject Alternative Names (SAN) properties have the IP address and the DNS of the machine where the component is running. This is needed for hostname verification to succeed in the Mutual TLS handshake process. Refer to [Enabling Transport Layer Security (TLS) during TO2](#enabling-transport-layer-security-tls-during-to2) for steps to create such keystore files.

Create a truststore file as follows:

```
$ keytool –import –file path/to/certificate –alias sampleCA –keystore path/to/truststore
```
Based on the component interactions, the truststore for each component must contain the following certificate entries:

```
OCS truststore contains entries to accept certificates of OPS and To0Scheduler.
OPS truststore contains entries to accept certifcates of OCS.
To0Scheduler truststore contains entries to accept certifcates of OCS.
```

 For information on the properties that needs to be updated for using the newly created files, refer to the following sections:  
 - [Setting up Resources for OCS](#setting-up-resources-for-ocs)  
 - [Running the Owner Companion Service with HTTPS](#running-the-owner-companion-service-with-https)  
 - [Running the To0Scheduler with HTTPS for T00](#running-the-to0scheduler-with-https-for-t00)  

Common Issues While Running the Demo
-------------------------------------

1.  ***Signature is Invalid during TO0***:

 This error occurs when the owner's certificate (and its public key) residing at the OCS in the owner keystore, does not match the public key present in the Ownership Voucher. To rectify this error, insert the owner key-pair at
 `<sdo-iot-platform-sdk-root>/demo/ocs/config/db/v1/creds/owner-keystore.p12`
per the steps outlined [here](#inserting-an-existing-owners-certificate-and-private-key-into-a-keystore). Refer to the properties file for details on properties to be updated.  The following is a sample error log at To0Scheduler:

```
o.s.i.t.t.To0SignatureServiceFactoryImpl.lambda$sign$0 - Obtaining signature from OCS
o.s.i.t.t.RestClient.signatureOperation - Error occurred while getting the signature for 59ec2905-e715-4bec-ad8f-ad5610d64895. 500
o.s.i.t.t.To0SignatureServiceFactoryImpl.lambda$sign$0 - Unable to get signature for 59ec2905-e715-4bec-ad8f-ad5610d64895
o.s.i.t.t.To0ClientSession.run - <POST http://10.0.0.1:8001/mp/113/msg/22,{"to0d":………
o.s.i.t.t.To0ScheduledClientSession.call - {"ec":3,"emsg":255,"em":"Signature of owner message is invalid."}
c.i.s.o.t.To0SchedulerEventsImpl.onFailure - TO0 failed for the device having uuid 59ec2905-e715-4bec-ad8f-ad5610d64895.

```
