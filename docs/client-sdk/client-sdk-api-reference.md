## SDO Client SDK API Reference
This section describes the APIs provided by the SDO Client SDK as well as public data structures required to support them.

## Data Structures
These structures are defined in the SDO Client SDK header file and used by various SDK APIs.

### Generic API Return Status
This type is returned by all APIs and indicates whether the API call succeeded or failed.

_**Syntax**_  
```
typedef enum {
    SDO_SUCCESS = 0,
    SDO_INVALID_PATH,
    SDO_CONFIG_NOT_FOUND,
    SDO_INVALID_STATE,
    SDO_RESALE_NOT_SUPPORTED,
    SDO_RESALE_NOT_READY,
    SDO_WARNING,
    SDO_ERROR,
    SDO_ABORT
} sdoSdkStatus;
```
_**Members**_  

**SDO_SUCCESS**  
        The API call succeeded.  
**SDO_INVALID_PATH**  
        A path or address specified was not found or not accessible.  
**SDO_CONFIG_NOT_FOUND**  
        An expected configuration file was not found or not accessible.  
**SDO_INVALID_STATE**  
        The SDK is in an invalid state to perform the requested operation.  
**SDO_RESALE_NOT_SUPPORTED**  
        This configuration of the SDK does not support the Resale Protocol.  
**SDO_RESALE_NOT_READY**  
        The SDK is not in a state to execute the Resale Protocol. This error will occur when the Resale API (sdoSdkResale) is called when device ownership transfer has not yet completed successfully.  
**SDO_WARNING**  
        This value is used in the SDK error callback to notify the Application that a transient failure occurred. See Error Handling Callback for details.  
**SDO_ERROR**  
        The API call did not succeed. We might extend this later (TBD) to allow returning of specific error types as positive values (for example, memory allocation failure, communications failure, and so on).   
**SDO_ABORT**  
        This value is returned by the error callback function to prevent the SDK from continuing the transfer of ownership protocol if an error occurred. Details are provided in Error Handling Callback.  

### SDO Device Status
This type indicates the current SDO protocol status of the device.

_**Syntax**_    
```
typedef enum {
    SDO_STATE_PRE_DI = 2,
    SDO_STATE_PRE_TO1,
    SDO_STATE_IDLE,
    SDO_STATE_RESALE,
    SDO_STATE_ERROR
} sdoSdkDeviceStatus;
```
_**Members**_  

**SDO_STATE_PRE_DI**  
        The SDK is in the pre-Device Initialization state. It is ready to run the DI protocol which can be initiated by calling the `sdoSdkRun()` API.  
**SDO_STATE_PRE_TO1**  
        The SDK has completed the DI stage and is ready for Device onboarding. The SDK will run the TO1 protocol if the `sdoSdkRun()` API is called.  
**SDO_STATE_IDLE**  
        The SDK has completed ownership transfer and is in the idle state. Calling `sdoSdkRun()` will have no effect. The Application may only call the `sdoSdkResale()` API to initiate the Resale protocol at his point.  
**SDO_STATE_RESALE**  
        The SDK has is now ready for resale. Calling `sdoSdkRun()` will run the TO1 & TO2 protocols, to carryout onboarding to the new (post-resale) device owner.  
**SDO_STATE_ERROR**  
        This API failed due to an internal error.  

### SDO SDK Error Values
This type is passed from the SDK to the Application when an error occurs and indicates details of the error. It is used by the error callback – see Error Handling Callback.  

_**Syntax**_  
```
typedef enum {
    SDO_RV_TIMEOUT = 1,
    SDO_CONN_TIMEOUT,
    SDO_DI_ERROR,
    SDO_TO1_ERROR,
    SDO_TO2_ERROR
} sdoSdkError;
```
_**Members**_  

**SDO_RV_TIMEOUT**  
        A timeout occurred when trying to contact the Rendezvous Server.  
**SDO_CONN_TIMEOUT**  
        A connection to either the Rendezvous or Owner Server timed out.  
**SDO_DI_ERROR**  
        A generic error occurred during the DI stage.  
**SDO_TO1_ERROR**  
        A generic error occurred during the TO1 protocol stage.  
**SDO_TO2_ERROR**  
        A generic error occurred during the TO2 protocol stage.  

### SDO Service Callback Type
This type value indicates the type of function the module callback must perform. These values are used by the Service Information Module Callback function.  

_**Syntax**_  
```
typedef enum {
    SDO_SI_START,
    SDO_SI_GET_DSI_COUNT,
    SDO_SI_SET_PSI,
    SDO_SI_GET_DSI,
    SDO_SI_SET_OSI,
    SDO_SI_END,
    SDO_SI_FAILURE
} sdoSdkSiType;
```
_**Members**_  

Below are the parameters of the Service Information Module Callback function described in Service Information Module Callback function.  

**SDO_SI_START**  
        This value indicates that the SDK is starting Service Info rounds. The module may perform pre-preparation operations at this time. The count and si parameters will be NULL.  
**SDO_SI_SET_PSI**  
        This value indicates that the SDK is returning a pre-service information (PSI) key-value pair that was received from the server to the module. The `si.key` contains the received key name and is a `NULL` terminated ASCII string. The `si.value` parameter contains the PSI value and is assumed to be a `NULL` terminated string. If no value was received, this value will by `NULL`. The `count` will contain the ‘index’ of the invocation, starting from 0 and incrementing on each subsequent call. If the module needs to retain the contents of the `si.key` or `si.value` parameters after the call returns, it must make an internal, local copy.  
**SDO_SI_GET_DSI_COUNT**  
        This value indicates that the SDK is requesting the module to return the total number of Device Service Info (DSI) strings it needs to send to the Owner Server. The maximum size of a string is 1,200 bytes. Strings larger than 1,200 bytes need to be split across multiple strings and this count must be factored into the total number of DSI strings for the module. On receiving this call, the module must place the number of DSI strings the module will report into the `*count` parameter. The `si` pointer will be `NULL`.  
**SDO_SI_GET_DSI**  
        This value indicates that the SDK is requesting the module to return a particular Device Service Info string. The count parameter contains the index of the DSI value requested. It will progress from zero to the value returned by the response to the `SDO_SI_GET_DSI_COUNT` callback minus 1. A module must use this value to track where in the DSI list it currently is. The module must set prepare two `NULL` terminated strings – one for the DSI key and another for the DSI value. The `si.key` and `si.value` pointers must point at these strings before returning from the callback. The memory allocated for si.key and si.value must remain valid (that is, not be freed) until the next time this callback is invoked – the service info callback sequence will end with `SDO_SI_END or SDO_SI_ERROR`.  
**SDO_SI_SET_OSI**  
        This value indicates that the SDK is providing a valid Owner Service Info (OSI) key-value pair to the module and it must process the provide OSI information. The count parameter is a progressively increasing index value of the provided OSI. The `si.key` will contain the key value and the `si.value` will contain the value of the string. Both are valid, `NULL` terminated strings. The value might be Base 64 encoded binary data. This encoding is module-dependent and should be indicated by the `si.key` contents.  
**SDO_SI_END**  
        This value indicates that the SDK has completed all Service Info rounds and the module can perform cleanup or final operations if required (like saving a file to disk, and so on) The `count` and the `si` parameters will be `NULL`.  
**SDO_SI_FAILURE**  
        This value indicates that an error occurred and the SDK is aborting or abandoning this Service Info round. The module must ignore all the information it received thus far (if any) and reset to its pre-Service Info state. The `count` and `si` parameters will be NULL.  

### Service Information Key Value Pair
This type contains a key name string and associated value string. Both items are pointers to ASCII NULL terminated strings.   

_**Syntax**_  
```
typedef struct sdoSdkSiKeyValue {
    char *key;
    char *value;
} sdoSdkSiKeyValue;
```
_**Members**_  

**key**  
        `NULL` terminated string containing the Key Name.  
**value**  
        `NULL` terminated string containing the Value associated with the Key Name. Binary values must be encoded in Base-64 format.  

### Service Information Module Callback
This type is a pointer to a callback function that is used to process Owner Service Information messages received from the Owner Server for a specific Service Information module. See Service Information Module Description for how it is used.

This callback function is invoked in the context of the executing onboarding protocol hence, although there is no fixed timeline, the module must complete execution in the shortest possible time.  

_**Syntax**_  

```
typedef int (*sdoSdkServiceInfoCB)(
    sdoSdkSiType     type,
    int              *count,
    sdoSdkSiKeyValue *si
);
```  
  
_**Parameters**_  

**type**  
This indicates the type of the callback as specified in the SDO Service Callback Type section.  
**count**  
An integer pointer that is used for multiple purposes. In the case of `SDO_SI_GET_DSI_COUNT`, the module must return the number of DSI entries it will send. For `SDO_SET_PSI, SDO_SI_GET_DSI` and `SDO_SI_SET_OSI`, it contains an index value into the PSI, DSI, and OSI lists respectively.  
**si**  
A pointer to a service info key-value pair of type `sdoSdkSiKeyValue` that contains a key and its corresponding value, both of which are `NULL` terminated strings.

_**Return Value**_  

The return value could be one of the following:  

-	`SDO_SI_CONTENT_ERROR`: Indicates that the module could not process the Service Information due to invalid content of either the key or value. An error will cause the onboarding protocol to fail and be retried.  
-	`SDO_SI_INTERNAL_ERROR`: Indicates that the module could not process the Service Information due to an internal error. An error will cause the onboarding protocol to fail and be retried.  
-	`SDO_SI_SUCCESS`: Indicates that the module was able to successfully process the Service Information (or ignored it successfully).
  

### Service Information Module Description
This structure describes a Service Information module that implements module functionality as described Section 2.3 and Section 2.5. Supported modules are known to the Application ahead of time and each module must have one Service Information Module entry, which is passed to the SDK in the `sdoSdkInit()` API.  

_**Syntax**_  
```
typedef struct {
    char                moduleName[16];
    sdoSdkServiceInfoCB serviceInfoCallback;
} sdoSdkServiceInfoModule;
```
_**Members**_  

**moduleName**  
        The symbolic name of the Service Information Module. This should be a NULL terminated string, no larger than 15 characters.  
**serviceInfoCallback**  
        This callback function will be invoked by the SDK to obtain Device Service Info from the module as well as pass Pre-Service Info and Owner Service Info to the module. This callback will be executed in the context of the onboarding protocol. See Service Information Module Callback.  

### Error Handling Callback
This type is a pointer to a callback function that is used to process errors during protocol execution. The Application can use information provided by this callback to perform application-specific operations. The Application can also control the execution of the protocol state machine by return different values as specified below.  

_**Syntax**_  
```
typedef int (*sdoSdkErrorCB)(
    sdoSdkStatus type,
    sdoSdkError  errorCode
);
```  
  
_**Parameters**_  

**type**  
This value specifies the type of error that occurred. It will be one of the following values (other `sdoSdkStatus` values are not used here):  
* `SDO_ERROR`: This indicates an unrecoverable error occurred. The SDK will continue with protocol restart for these types of errors but it is unlikely that the operation will succeed. It is advisable to abort the operation and retry later.  
* `SDO_WARNING`: This indicates that a transient error occurred. The SDK will continue with protocol restart, which might fix the problem. It is advisable that the Application allows the restart to take place.  
**errorCode**  
This value indicates details of the error that occurred. See description in SDO SDK Error Values.  

_**Return Value**_  

The return value could be one of the following constants (***Note:*** These values are constants that are defined in the SDO SDK header file):

* `SDO_SUCCESS`: Indicates that the error was handled and the SDK should continue with its recovery or restart as required.  
* `SDO_ABORT`: This causes the SDK to terminate protocol processing and return to the caller (such as, the `sdoSdkRun()` API returns). The Application can re-invoke this API later to re-initiate the SDO onboarding process.


## SDK API Functions  

The following functions are provided by the SDK and defined in the SDK API header file.

### Initialize SDO SDK

The Application must invoke this API before any other APIs since this API initializes all internal data and state of the SDK.  

_**Syntax**_  
```
sdoSdkStatus sdoSdkInit(
    sdoSdkErrorCB    *errorHandlingCallback,
    uint32_t          numModules,
    sdoSdkModuleInfo *moduleInformation,
);
```  

_**Parameters**_ 	

**errorHandlingCallback**  

This is the Application’s error handling function and will be called by the SDK when an error is encountered. This value can be `NULL` in which case, errors will not be reported to the Application, and the SDK will take the appropriate recovery and/or restart action as required.  

***Note:*** Passing `NULL` might cause the SDK to remain in an infinite loop until the onboarding process completes successfully.  

**numModules**  
Number of Service Information modules contained in the following `moduleInformation` list parameter. If no Application-specific modules are available, this value should be zero.  
**moduleInformation**  
Service Module Information description for each available Service Information module as described in Service Information Module Description. If no Application-specific modules are available, this value should be `NULL`.  

_**Return Value**_  

This function returns success or an error code as defined in Generic API Return Status.
 
### Get Current SDO SDK Status
This function returns the current state of the SDO SDK. It may only be called after the SDK has been initialized using the `sdoSdkInit()` API.  

_**Syntax**_  
`sdoSdkDeviceState sdoSdkGetStatus(void);`  

_**Return Value**_    

This function returns a value of type sdoSdkDeviceStatus as described in SDO Device Status.

### Execute SDO SDK Onboarding Protocol
The Application invokes this API to begin the onboarding process that is, TO1. The onboarding process has completed successfully when this function returns `SDO_SUCCESS`. If this API returns an error, the Application may retry the onboarding process by calling this API again immediately or after a sleep/reset cycle as determined by the use case.
The SDK will invoke the Application error callback if an error occurs in this phase. Additionally, module-specific callbacks will be invoked when Service Information is received from the Owner Server during the TO2 stage. These callbacks are invoked in the context of the callers thread and the callbacks must not call any SDK APIs since the SDK is not yet re-entrant. 

_**Syntax**_  
`sdoSdkStatus sdoSdkRun(void);`  

_**Return Value**_  

This function returns success or an error code as defined in Generic API Return Status.

### Prepare the SDO SDK for Resale
The Application invokes this API to prepare the device for resale. The SDK marks internal state to pre-TO1 and returns. After preparing the SDK for resale, a subsequent call to the `sdoSdkRun()` API will cause the SDK to perform the TO1 stage again, with new owner credentials that were updated at the end of the previous TO2 protocol stage.  

_**Syntax**_    
`sdoSdkStatus sdoSdkResale(void);`  

_**Return Value**_    

This function returns success or an error code as defined in Generic API Return Status  
