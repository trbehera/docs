This section defines the “system” module (fdo_sys) specification which provides basic onboarding services
for FDO capable devices. A module is a set of key-value pairs; they define the onboarding operations
that can be performed on a given FDO device. Module key-value pairs are exchanged between the device
and it’s owning Device Management Service. It is up to the owning management service and the
device to interpret the key-value pairs in accordance with the module specification.

## fdo_sys module definition
The system module defines basic onboarding operations such as downloading content and executing
commands. The following table describes key-value pairs for the system module.

| | | |
|-|-|-|
| **Key Name** | **Value** | **Meaning** |
| fdo_sys:active | CBOR Bool | True to activate the fdo_sys module. |
| fdo_sys:exec | CBOR array of string command arguments. | Performs a system call. |
| fdo_sys:filedesc | CBOR String | Describes a path to a file that will be used in subsequent file operations. |
| fdo_sys:write | CBOR BSTR | Appends a block of data to the current file description. |

### fdo_sys:exec message

The fdo_sys:exec commands performs a system call on the device. The value of this command is a
CBOR Array of string values. The first string in the array is the command to execute and the
remaining strings are the arguments to the command. It is expected that it would use “exec”
(system call) or similar API to execute the command on the device.

| | |
|-|-|
| JSON (readable description) | ["/bin/sh","startup.sh"] |
| CBOR | `82` # array(2) <br> `67` # text(7) <br> `2F62696E2F7368` # "/bin/sh" <br> `6A` # text(10) <br> `737461727475702E7368` # "startup.sh"|
| C example | execvp(“/bin/sh”,(char* []) {“shartup.sh”}) |

### fdo_sys:filedesc message

The fdo_sys:filedesc command describes the path to a file the will be used as a part of the
on-boarding process. A zero-length file is expected to exist on the local file system after this
command is received. If the described file already exists it is truncated to zero length, otherwise
a zero-length file is created. The permissions for the created file are set to read/write
for the user account the module is running under. File permissions can subsequently be modified with
the fdo_sys:exec command if needed. If a path is not included as a part of the file name, the
current working directory of the module is assumed. All subsequent fdo_sys:write operations will
append to this file. If another sdo_sys.filedesc message is received then it replaces the current
and all subsequent fdo_sys:write messages will start appending to the new current filedesc.

### fdo_sys:write message
Write an array of bytes to the end of the file described by the last fdo_sys:filedesc command.
Since fdo_sys:filedesc creates a zero length file the first write will be at the beginning of the
file and all subsequent writes will be at the end. If this message is sent without being preceded
by sdo_sys.filedesc then an message `255: INVALID_MESSAGE_ERROR` will be thrown and TO2 will not
complete. Once a fdo_sys:filedesc has been received, many fdo_sys:write messages can follow.

## Examples
Below are examples of the fdo_sys messages encoded as CBOR. The JSON examples are just human
readable definitions while the actual messages are always CBOR. The encoding includes the entire
TO2.OwnerServiceInfo message include the isMore and isDone flags.

Device should advertise its supports fdo_sys.

***Note***: This example just includes the module list key value pairs and not all the required values for the devmod.

### DeviceServiceInfo (devmod)

| | | |
|-|-|-|
| Diagnostic Notation (not used by protocol) | [false, [[["devmod:active", "true"], ["devmod:nummodules", 1], ["devmod:modules", [1, 1, "fdo_sys"]]]]] | |
| CBOR | 82<br>F4<br>81<br>83<br>82<br>6D<br>6465766D6F643A616374697665<br>64<br>74727565<br>82<br>71<br>6465766D6F643A6E756D6D6F64756C6573<br>01<br>82<br>6E<br>6465766D6F643A6D6F64756C6573<br>83<br>01<br>01<br>69<br>66646F5F7379732D31 | array(2) <br> primitive(20) <br> array(1) <br> array(3) <br> array(2) <br> text(13) <br> "devmod:active" <br> text(4) <br> "true" <br> array(2) <br> text(17) <br> "devmod:nummodules" <br> unsigned(1) <br> array(2) <br> text(14) <br> "devmod:modules" <br> array(3) <br> unsigned(1) <br> unsigned(1) <br> text(9) <br> "fdo_sys" |

### OwnerServiceInfo

#### fdo_sys:active
| | | |
|-|-|-|
| Diagnostic Notation (not used by protocol) | [true, false, [[["fdo_sys:active", true]]]] | |
| CBOR | 83 <br> F5 <br> F4 <br> 81 <br> 81 <br> 82 <br> 6E <br> 66646F5F7379733A61637469766 <br> F5 | array(3)  <br> primitive(21) <br> primitive(20) <br> array(1) <br> array(1) <br> array(2) <br> text(14) <br> "fdo_sys:active" <br> primitive(21) |

#### fdo_sys:filedesc
| | | |
|-|-|-|
| Diagnostic Notation (not used by protocol) | [true, false, [[["fdo_sys:filedesc", "startup.bat"]]]] | |
| CBOR | 83 <br> F5 <br> F4 <br> 81 <br> 81 <br> 82 <br> 70 <br> 66646F5F7379733A66696C6564657363 <br> 6B <br> 737461727475702E626174 | array(3) <br> primitive(21) <br> primitive(20) <br> array(1) <br> array(1) <br> array(2) <br> text(16) <br> "fdo_sys:filedesc" <br> text(11) <br> "startup.bat" |

#### fdo_sys:write
| | | |
|-|-|-|
| Diagnostic Notation (not used by protocol) | [true, false, [[["fdo_sys:write", h'4045…']]]] | |
| CBOR | 83 <br> F5 <br> F4 <br> 81 <br> 81 <br> 82 <br> 6D <br> 66646F5F7379733A7772697465 <br> 59 0116 <br> 4045.. | array(3) <br> primitive(21) <br> primitive(20) <br> array(1) <br> array(1) <br> array(2) <br> text(13) <br> "fdo_sys:write" <br> bytes(278) <br> rest of 278 byte file |

#### fdo_sys:exec
| | | |
|-|-|-|
| Diagnostic Notation (not used by protocol) | [false, true, [[["fdo_sys:exec", ["cmd", "/c", "startup.bat"]]]] | |
| CBOR | 83 <br> F4 <br> F5 <br> 81 <br> 81 <br> 82 <br> 6C <br> 66646F5F7379733A65786563 <br> 83 <br> 63 <br> 636D64 <br> 62 <br> 2F63 <br> 6B <br> 737461727475702E626174 | array(3) <br> primitive(20) <br> primitive(21) <br> array(1) <br> array(1) <br> array(2) <br> text(12) <br> "fdo_sys:exec" <br> array(3) <br> text(3) <br> "cmd" <br> text(2) <br> "/c" <br> text(11) <br> "startup.bat" |

