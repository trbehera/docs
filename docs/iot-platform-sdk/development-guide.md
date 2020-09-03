The IOT Platform SDK consists of Owner Companion Service (OCS) that can be implemented to integrate any other device management solution.

Both the OPS and To0Scheduler are bound to the OCS by a set of pre-defined contracts, as specified in the module `libocs`, that can be implemented to perform certain pre-defined tasks. The contract is backed by a set of REST APIs/interfaces, where each individual REST API/interface is tied to exactly one method from the pre-defined contract. Each REST API/interface carries out only the corresponding task, and accepts/returns only the corresponding Message Type object.

The implementation of these REST contracts in an OCS can be done in any programming language. A sample java-based reference implementation of OCS is provided by the module `ocsfs` that uses file-system as database. As such, both OPS and To0Scheduler will work out-of-the-box with any implementation of OCS that supports the pre-defined REST contracts.

!!! Note
    The implementation of the pre-defined contracts can be backed by non-REST interfaces as well. However, this would invlove updates to both OPS and To0Scheduler to facilitate communication with the newly defined interface at OCS. Such a change would involve, but is not limited to, updates to existing class `RestClient.java` at both OPS and To0Scheduler.

## Rest Contracts between OCS (server) and OPS/To0Scheduler (client)

Following are the pre-defined REST API specifications for all the resource paths that must be implemented and exposed by an OCS implementation. Both, OPS and To0Scheduler make requests to the OCS implementation, as clients, using this format.

| Operation                               | Description                                                  | Path/Query Parameters                    | Request Body  | Response Body |
| ---------------------------------------:|:------------------------------------------------------------:|:----------------------------------------:|:-------------:|--------------:|
| `GET` /v1/devices/{deviceId}/voucher    | Get the ownership voucher corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier      |               | OwnerVoucher  |
| `POST` /v1/devices/voucher              | Store the ownership voucher.                                 |                                          | OwnerVoucher  |               |
| `GET` /v1/devices/{deviceId}/state      | Get the state information corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier      |               | DeviceState   |
| `POST` /v1/devices/{deviceId}/state     | Store the state information corresponding to the `deviceId`. | Path: `deviceId`: Device identifier      | DeviceState   |               |
| `GET` /v1/devices/{deviceId}/msgs       | Get service info array corresponding to the `deviceId`.      | Path: `deviceId`: Device identifier      |               | [SviMessage]  |
| `POST` /v1/devices/{deviceId}/msgs      | Store service info array corresponding to the `deviceId`.    | Path: `deviceId`: Device identifier      | [SviMessage]  |               |
| `PUT` /v1/devices/{deviceId}/msgs       | Store module message info corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier      | ModuleMessage |               |
| `DELETE` /v1/devices/{deviceId}/msgs    | Delete service info array corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier      |               |               |
| `GET` /v1/devices/{deviceId}/psi        | Get module message array corresponding to the `deviceId`.    | Path: `deviceId`: Device identifier      |               |[ModuleMessage]|
| `GET` /v1/ devices/{deviceId}/values/{valueId}  | Get the serviceinfo value corresponding to the `valueId` for the device identified by `deviceId`.   | Path: `deviceId`: Device identifier, `valueId`: Serviceinfo resource identifier <br> Query `start`: start index, `end`: end index  |               | Byte Array  |
| `PUT` /v1/ devices/{deviceId}/values/{valueId}  | Store the serviceinfo value corresponding to the `valueId` for the device identified by `deviceId`. | Path: `deviceId`: Device identifier, `valueId`: Serviceinfo resource identifier  |               | Byte Array    |               |
| `DELETE` /v1/ devices/{deviceId}/values/{valueId}  | Delete the serviceinfo value corresponding to the `valueId` for the device identified by `deviceId`.  | Path: `deviceId`: Device identifier, `valueId`: Serviceinfo resource identifier  |               |               |               |
| `GET` /v1/devices/{deviceId}/setupinfo  | Get new setup information corresponding to the `deviceId`.  | Path: `deviceId`: Device identifier       |               | SetupInfoResponse |
| `POST` /v1/devices/{deviceId}/errors    | Store the error information corresponding to the `deviceId`. | Path: `deviceId`: Device identifier      | DeviceState   |                   |
| `POST` /v1/signatures/{deviceId}        | Generate signature of input data and return it along with associated info, corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier |               |  String        | SignatureResponse |
| `POST` /v1/ciphers/{deviceId}           | Perform the `operation` and return data corresponding to the .  | Path: `deviceId`: Device identifier; Query: `operation`: encipher/decipher | Byte Array   | Byte Array    |
| `DELETE` /devices/{deviceId}/blob       | Delete the `deviceId` along with all the associated data.    | Path: `deviceId`: Device identifier      |                |               |
| `GET` /v1/devices/{deviceId}/sessioninfo  | Get the TO2 session info corresponding to the `deviceId`.  | Path: `deviceId`: Device identifier      |                | To2DeviceSessionInfo |
| `POST` /v1/devices/{deviceId}/sessioninfo  | Update the TO2 session info corresponding to the `deviceId`. | Path: `deviceId`: Device identifier   | To2DeviceSessionInfo |               |
| `DELETE` /v1/devices/{deviceId}/sessioninfo  | Delete the TO2 session info corresponding to the `deviceId`.   | Path: `deviceId`: Device identifier |               |               |
| `GET` /devices/{deviceId}/resale  | Get the `resale` flag indicating owner's support for resale of the corresponding `deviceId`.   | Path: `deviceId`: Device identifier |               |    Boolean    |

## Rest Contracts between To0Scheduler (server) and OCS (client)

An OCS implementation must, also, act as a client to trigger TO0 for set of devices, by making the following request to To0Scheduler. To0Scheduler accepts the request from OCS to initiate TO0 for the list of devices.

| Operation                               | Description                                                  | Path/Query Parameters                    | Request Body  | Response Body |
| ---------------------------------------:|:------------------------------------------------------------:|:----------------------------------------:|:-------------:|--------------:|
| `POST` /v1/to0/devices                  | Trigger TO0 for an array of devices.                         |                                          | To0Request    |               |


## Message Types

Following is a list of message types that are sent in the message body of each request/response. Each message type follows the standard JSON* schema. Statements after '#' inside the JavaScript Object Notation (JSON), represents the purpose of the field.

### DeviceState
This JSON* structure represents the state information of the device.

*Message Body:*
```
{
  "to2Error": ProtocolError Object,    # TO2 failure information
  "to2Timestamp": String,              # Successful TO2 completion time
  "g3": String,                        # Device Identifier/GUID
  "to0Error": String,                  # TO0 failure information
  "to0Timestamp": String,              # Successful TO0 completion time
  "to0Ws": Integer,                    # Number of seconds for which last successul TO0 is valid.
  "to2State": String                   # TO2 state
}
```
Every field is optional.


### *ProtocolError*
This JSON* structure represents the device's error information that occurred during execution of the protocol.

*Message Body:*
```
{
  "ec": Integer,    # Error code
  "emsg": Integer,  # Message ID
  "em": String      # Error message
}
```


### *ModuleMessage*
This JSON* structure contains the key-value pairs for a particular module name. It is used to store both the device's serviceinfo and the owner's pre-serviceinfo.

*Message Body:*
```
{
  "module": String,    # module name
  "msg": String,       # (pre-)serviceinfo key
  "value":  String     # (pre-)serviceinfo value
}
```


### *SviMessage*
This JSON* structure represents information about the owner service info.

*Message Body:*
```
{
  "module":  String,     # Module name
  "msg": String,         # Message to be sent
  "valueLen": Integer,   # Length of the value of the message
  "valueId": String,     # Key to use when retrieving the serviceinfo value
  "enc": String          # Encoding expected to use when sending the value
}
```


### *SetupInfoResponse*
This JSON* structure represents the new Device identifier and the Rendezvous information.

*Message Body:*
```
{
  "g3": String,                             # New device identifier/GUID
  "r3": [
          RendezvousInstruction Object      # Array of RendezvousInstruction objects
        ]
}
```


### *RendezvousInstruction*
This JSON* structure represents the rendezvous information. Please refer to the [RendezvousInfo](../protocol-specification/protocol-data-types.md#rendezvousinfo) to know about these values.

*Message Body:*
```
{
  "only": String,
  "ip": String,
  "po": Integer,
  "pow": Integer,
  "dn": String,
  "sch": String,
  "cch": String,
  "ui": Integer,
  "ss": String,
  "pw": String,
  "wsp": String,
  "me": String,
  "pr": String,
  "delaysec": Integer
}
```


### *SignatureResponse*
This JSON* structure represents the response to the signature operation.

*Message Body:*
```
{
  "sg": String,    # Base-64 encoded signature information
  "pk": String,    # Base-64 encode public key information that can verify the signature
  "alg": String    # Algorithm of the public key
}
```


### *OwnerVoucher*
This JSON* structure represents the Ownership voucher of the device. Please refer to the [Ownership Voucher](../protocol-specification/detailed-protocol-description.md#pmownershipproxy-type-3) about the structure.


### *Message41Store*
This JSON* structure represents the information from Secure Device Onboard protocol's request [TO2.HelloDevice, Type 40](../protocol-specification/detailed-protocol-description.md#to2hellodevice-type-40) and its subsequent response [TO2.ProveOPHdr, Type 41](../protocol-specification/detailed-protocol-description.md#to2proveophdr-type-41). This information is stored as a part of TO2 session information per device.

*Message Body:*
```
{
  "n6": String,                  # Nonce n6
  "kx": String,                  # Key exchange information
  "ownershipVoucher": String,    # ownership voucher
  "cs": String,                  # cipher suite
  "kxEcdhPublicKey": String,     # Public key being used for ECDH key exchange
  "kxEcdhPrivateKey": String,    # Private key being used for ECDH key exchange
  "kxEcdhRandom": String,        # Random being used for ECDH key exchange
  "kxDhPublicKey": String,       # Public key being used for DH key exchange
  "kxDhPrivateKey": String,      # Public key being used for DH key exchange
  "asymRandom": String           # Random being used for Asymmetric key exchange
}
```


### *Message45Store*
This JSON* structure represents the information from Secure Device Onboard protocol's request [TO2.ProveDevice, Type 44](../protocol-specification/detailed-protocol-description.md#to2provedevice-type-44) and its subsequent response [TO2.GetNextDeviceServiceInfo, Type 45](../protocol-specification/detailed-protocol-description.md#to2getnextdeviceserviceinfo-type-45). This information is stored as a part of TO2 session information per device.

*Message Body:*
```
{
  "n7": String,    # Nonce n7
  "nn": Integer,   # Total number of serviceinfo
  "xb": String     # Second part of key-exchange
}
```


### *Message47Store*
This JSON* structure contains the new ownership voucher information, temporarily, until the end of TO2 protocol, created during response generation of [TO2.SetupDevice, Type 47](../protocol-specification/detailed-protocol-description.md#to2setupdevice-type-47). This information is stored as a part of TO2 session information per device.

*Message Body:*
```
{
  "newOwnershipVoucher": String    # the new ownership voucher structure without hmac
}
```


### *DeviceCryptoInfo*
This JSON* structure contains the nonce and counter of the Initialization Vector for the Secure Device Onboard TO2 cipher mode of operation.

*Message Body:*
```
{
  "ctrNonce": String,    # Nonce used for CTR mode
  "ctrCounter": Long     # Counter value used for CTR mode
}
```


### *To2DeviceSessionInfo*
This JSON* structure represents the complete TO2 session information. It's a combination of multiple structures, where each structure is either null or contains values. If any of the structure is null, OCS treats the structure as non-updatable in its store, while OPS considers the same to be an erroneous case. If any of the structure contains appropriate information, OCS overrides the current content with the new content, while OPS processes the content during TO2 protocol execution.

*Message Body:*
```
{
  "messsage41Store": Message41Store Object,
  "messsage45Store": Message45Store Object,
  "messsage47Store": Message47Store Object,
  "deviceCryptoInfo": DeviceCryptoInfo Object
}
```


### *To0Request*
This JSON* structure represents the request message that is sent from the OCS to schedule an array of devices for TO0.

*Message Body:*
```
{
  "guids": [
             String            # Array of Device Identifiers/GUID
           ],
  "waitSeconds": String        # Suggested number of seconds for which TO0 will be valid
}
```