## FIDO Device Onboard (FDO) Client SDK API Reference
This section describes the APIs provided by the FDO Client SDK as well as public data structures required to support them.

## Data Structures
These structures are defined in the FDO Client SDK header file and used by various SDK APIs.

### Generic API Return Status
This type is returned by all APIs and indicates whether the API call succeeded or failed.

_**Syntax**_  
```
typedef enum {
    FDO_SUCCESS = 0,
    FDO_INVALID_PATH,
    FDO_CONFIG_NOT_FOUND,
    FDO_INVALID_STATE,
    FDO_RESALE_NOT_SUPPORTED,
    FDO_RESALE_NOT_READY,
    FDO_WARNING,
    FDO_ERROR,
    FDO_ABORT
} fdoSdkStatus;
```
_**Members**_  

**FDO_SUCCESS**  
        The API call succeeded.  
**FDO_INVALID_PATH**  
        A path or address specified was not found or not accessible.  
**FDO_CONFIG_NOT_FOUND**  
        An expected configuration file was not found or not accessible.  
**FDO_INVALID_STATE**  
        The SDK is in an invalid state to perform the requested operation.  
**FDO_RESALE_NOT_SUPPORTED**  
        This configuration of the SDK does not support the Resale Protocol.  
**FDO_RESALE_NOT_READY**  
        The SDK is not in a state to execute the Resale Protocol. This error will occur when the Resale API (fdoSdkResale) is called when device ownership transfer has not yet completed successfully.  
**FDO_WARNING**  
        This value is used in the SDK error callback to notify the Application that a transient failure occurred. See Error Handling Callback for details.  
**FDO_ERROR**  
        The API call did not succeed. We might extend this later (TBD) to allow returning of specific error types as positive values (for example, memory allocation failure, communications failure, and so on).   
**FDO_ABORT**  
        This value is returned by the error callback function to prevent the SDK from continuing the transfer of ownership protocol if an error occurred. Details are provided in Error Handling Callback.  

### FDO Device Status
This type indicates the current FDO protocol status of the device.

_**Syntax**_    
```
typedef enum {
    FDO_STATE_PRE_DI = 2,
    FDO_STATE_PRE_TO1,
    FDO_STATE_IDLE,
    FDO_STATE_RESALE,
    FDO_STATE_ERROR
} fdoSdkDeviceStatus;
```
_**Members**_  

**FDO_STATE_PRE_DI**  
        The SDK is in the pre-Device Initialization state. It is ready to run the DI protocol which can be initiated by calling the `fdoSdkRun()` API.  
**FDO_STATE_PRE_TO1**  
        The SDK has completed the DI stage and is ready for Device onboarding. The SDK will run the TO1 protocol if the `fdoSdkRun()` API is called.  
**FDO_STATE_IDLE**  
        The SDK has completed ownership transfer and is in the idle state. Calling `fdoSdkRun()` will have no effect. The Application may only call the `fdoSdkResale()` API to initiate the Resale protocol at his point.  
**FDO_STATE_RESALE**  
        The SDK has is now ready for resale. Calling `fdoSdkRun()` will run the TO1 & TO2 protocols, to carryout onboarding to the new (post-resale) device owner.  
**FDO_STATE_ERROR**  
        This API failed due to an internal error.  

### FDO SDK Error Values
This type is passed from the SDK to the Application when an error occurs and indicates details of the error. It is used by the error callback – see Error Handling Callback.  

_**Syntax**_  
```
typedef enum {
    FDO_RV_TIMEOUT = 1,
    FDO_CONN_TIMEOUT,
    FDO_DI_ERROR,
    FDO_TO1_ERROR,
    FDO_TO2_ERROR
} fdoSdkError;
```
_**Members**_  

**FDO_RV_TIMEOUT**  
        A timeout occurred when trying to contact the Rendezvous Server.  
**FDO_CONN_TIMEOUT**  
        A connection to either the Rendezvous or Owner Server timed out.  
**FDO_DI_ERROR**  
        A generic error occurred during the DI stage.  
**FDO_TO1_ERROR**  
        A generic error occurred during the TO1 protocol stage.  
**FDO_TO2_ERROR**  
        A generic error occurred during the TO2 protocol stage.  

### FDO Service Callback Type
This type value indicates the type of function the module callback must perform. These values are used by the Service Information Module Callback function.  

_**Syntax**_  
```
typedef enum {
    FDO_SI_START,
    FDO_SI_GET_DSI_COUNT,
    FDO_SI_SET_PSI,
    FDO_SI_GET_DSI,
    FDO_SI_SET_OSI,
    FDO_SI_END,
    FDO_SI_FAILURE
} fdoSdkSiType;
```
_**Members**_  

Below are the parameters of the Service Information Module Callback function described in Service Information Module Callback function.  

**FDO_SI_START**  
        This value indicates that the SDK is starting Service Info rounds. The module may perform pre-preparation operations at this time. The count and si parameters will be NULL.  
**FDO_SI_SET_PSI**  
        This value indicates that the SDK is returning a pre-service information (PSI) key-value pair that was received from the server to the module. The `si.key` contains the received key name and is a `NULL` terminated ASCII string. The `si.value` parameter contains the PSI value and is assumed to be a `NULL` terminated string. If no value was received, this value will by `NULL`. The `count` will contain the ‘index’ of the invocation, starting from 0 and incrementing on each subsequent call. If the module needs to retain the contents of the `si.key` or `si.value` parameters after the call returns, it must make an internal, local copy.  
**FDO_SI_GET_DSI_COUNT**  
        This value indicates that the SDK is requesting the module to return the total number of Device Service Info (DSI) strings it needs to send to the Owner Server. The maximum size of a string is 1,200 bytes. Strings larger than 1,200 bytes need to be split across multiple strings and this count must be factored into the total number of DSI strings for the module. On receiving this call, the module must place the number of DSI strings the module will report into the `*count` parameter. The `si` pointer will be `NULL`.  
**FDO_SI_GET_DSI**  
        This value indicates that the SDK is requesting the module to return a particular Device Service Info string. The count parameter contains the index of the DSI value requested. It will progress from zero to the value returned by the response to the `FDO_SI_GET_DSI_COUNT` callback minus 1. A module must use this value to track where in the DSI list it currently is. The module must set prepare two `NULL` terminated strings – one for the DSI key and another for the DSI value. The `si.key` and `si.value` pointers must point at these strings before returning from the callback. The memory allocated for si.key and si.value must remain valid (that is, not be freed) until the next time this callback is invoked – the service info callback sequence will end with `FDO_SI_END or FDO_SI_ERROR`.  
**FDO_SI_SET_OSI**  
        This value indicates that the SDK is providing a valid Owner Service Info (OSI) key-value pair to the module and it must process the provide OSI information. The count parameter is a progressively increasing index value of the provided OSI. The `si.key` will contain the key value and the `si.value` will contain the value of the string. Both are valid, `NULL` terminated strings. The value might be Base 64 encoded binary data. This encoding is module-dependent and should be indicated by the `si.key` contents.  
**FDO_SI_END**  
        This value indicates that the SDK has completed all Service Info rounds and the module can perform cleanup or final operations if required (like saving a file to disk, and so on) The `count` and the `si` parameters will be `NULL`.  
**FDO_SI_FAILURE**  
        This value indicates that an error occurred and the SDK is aborting or abandoning this Service Info round. The module must ignore all the information it received thus far (if any) and reset to its pre-Service Info state. The `count` and `si` parameters will be NULL.  

### Service Information Key Value Pair
This type contains a key name string and associated value string. Both items are pointers to ASCII NULL terminated strings.   

_**Syntax**_  
```
typedef struct fdoSdkSiKeyValue {
    char *key;
    char *value;
} fdoSdkSiKeyValue;
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
typedef int (*fdoSdkServiceInfoCB)(
    fdoSdkSiType     type,
    int              *count,
    fdoSdkSiKeyValue *si
);
```  
  
_**Parameters**_  

**type**  
This indicates the type of the callback as specified in the FDO Service Callback Type section.  
**count**  
An integer pointer that is used for multiple purposes. In the case of `FDO_SI_GET_DSI_COUNT`, the module must return the number of DSI entries it will send. For `FDO_SET_PSI, FDO_SI_GET_DSI` and `FDO_SI_SET_OSI`, it contains an index value into the PSI, DSI, and OSI lists respectively.  
**si**  
A pointer to a service info key-value pair of type `fdoSdkSiKeyValue` that contains a key and its corresponding value, both of which are `NULL` terminated strings.

_**Return Value**_  

The return value could be one of the following:  

-	`FDO_SI_CONTENT_ERROR`: Indicates that the module could not process the Service Information due to invalid content of either the key or value. An error will cause the onboarding protocol to fail and be retried.  
-	`FDO_SI_INTERNAL_ERROR`: Indicates that the module could not process the Service Information due to an internal error. An error will cause the onboarding protocol to fail and be retried.  
-	`FDO_SI_SUCCESS`: Indicates that the module was able to successfully process the Service Information (or ignored it successfully).
  

### Service Information Module Description
This structure describes a Service Information module that implements module functionality as described Section 2.3 and Section 2.5. Supported modules are known to the Application ahead of time and each module must have one Service Information Module entry, which is passed to the SDK in the `fdoSdkInit()` API.  

_**Syntax**_  
```
typedef struct {
    char                moduleName[16];
    fdoSdkServiceInfoCB serviceInfoCallback;
} fdoSdkServiceInfoModule;
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
typedef int (*fdoSdkErrorCB)(
    fdoSdkStatus type,
    fdoSdkError  errorCode
);
```  
  
_**Parameters**_  

**type**  
This value specifies the type of error that occurred. It will be one of the following values (other `fdoSdkStatus` values are not used here):  
* `FDO_ERROR`: This indicates an unrecoverable error occurred. The SDK will continue with protocol restart for these types of errors but it is unlikely that the operation will succeed. It is advisable to abort the operation and retry later.  
* `FDO_WARNING`: This indicates that a transient error occurred. The SDK will continue with protocol restart, which might fix the problem. It is advisable that the Application allows the restart to take place.  
**errorCode**  
This value indicates details of the error that occurred. See description in FDO SDK Error Values.  

_**Return Value**_  

The return value could be one of the following constants (***Note:*** These values are constants that are defined in the FDO SDK header file):

* `FDO_SUCCESS`: Indicates that the error was handled and the SDK should continue with its recovery or restart as required.  
* `FDO_ABORT`: This causes the SDK to terminate protocol processing and return to the caller (such as, the `fdoSdkRun()` API returns). The Application can re-invoke this API later to re-initiate the FDO onboarding process.


## SDK API Functions  

The following functions are provided by the SDK and defined in the SDK API header file.

### Initialize FDO SDK

The Application must invoke this API before any other APIs since this API initializes all internal data and state of the SDK.  

_**Syntax**_  
```
fdoSdkStatus fdoSdkInit(
    fdoSdkErrorCB    *errorHandlingCallback,
    uint32_t          numModules,
    fdoSdkModuleInfo *moduleInformation,
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
 
### Get Current FDO SDK Status
This function returns the current state of the FDO SDK. It may only be called after the SDK has been initialized using the `fdoSdkInit()` API.  

_**Syntax**_  
`fdoSdkDeviceState fdoSdkGetStatus(void);`  

_**Return Value**_    

This function returns a value of type fdoSdkDeviceStatus as described in FDO Device Status.

### Execute FDO SDK Onboarding Protocol
The Application invokes this API to begin the onboarding process that is, TO1. The onboarding process has completed successfully when this function returns `FDO_SUCCESS`. If this API returns an error, the Application may retry the onboarding process by calling this API again immediately or after a sleep/reset cycle as determined by the use case.
The SDK will invoke the Application error callback if an error occurs in this phase. Additionally, module-specific callbacks will be invoked when Service Information is received from the Owner Server during the TO2 stage. These callbacks are invoked in the context of the callers thread and the callbacks must not call any SDK APIs since the SDK is not yet re-entrant. 

_**Syntax**_  
`fdoSdkStatus fdoSdkRun(void);`  

_**Return Value**_  

This function returns success or an error code as defined in Generic API Return Status.

### Prepare the FDO SDK for Resale
The Application invokes this API to prepare the device for resale. The SDK marks internal state to pre-TO1 and returns. After preparing the SDK for resale, a subsequent call to the `fdoSdkRun()` API will cause the SDK to perform the TO1 stage again, with new owner credentials that were updated at the end of the previous TO2 protocol stage.  

_**Syntax**_    
`fdoSdkStatus fdoSdkResale(void);`  

_**Return Value**_    

This function returns success or an error code as defined in Generic API Return Status  
