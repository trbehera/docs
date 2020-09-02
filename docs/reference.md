## Project Documentation

The [Introduction](index.md) provides an overview of Secure Device Onboard operation, its
associated protocols, and the software components that implement those protocols and support the
Secure Device Onboard ecosystem.

[All-in-one Demo](all-in-one-demo/introduction.md) documentation provided information on how to run
All-in-one Demo that facilitates an easy demonstration of Secure Device Onboard protocols.

[Client SDK](client-sdk/client-sdk-reference-guide.md) documentation provides information for
device manufacturers on how to use the Client SDK to support Secure Device Onboard on their devices.
The Client SDK is a portable implementation of the Secure Device Onboard protocol state machines,
cryptographic operations, and associated support software.

[IOT Platform SDK](iot-platform-sdk/introduction.md) documentation provides developers and system
integrators with information on how to use the IOT Platform SDK to integrate Secure Device Onboard
capabilities into their IOT Platform or Device Management System.

[Rendezvous Service](rendezvous-service/introduction.md) documentation provides information on how
to configure and operate the Rendezvous service component software.

[Manufacturing Tools](supply-chain-tools/manufacturing/manufacturer-enablement-guide.md)
documentation provides information for device manufactures on how to use the Manufacturing Toolkit
to integrate Secure Device Onboard into their manufacturing processes.

[Reseller Tools](supply-chain-tools/reseller/reseller-enablement-guide.md) documentation provides
information for resellers of enabled devices to use the Reseller Toolkit to prepare devices for
transfer of ownership to new entities.

[Key Store Setup](supply-chain-tools/keystore-guide.md) documentation provides manufacturers and
resellers with information on how to set up key-store capability for different Secure Device Onboard
components.

[Protocol Reference Implementation](protocol-reference-implementation/introduction.md) documentation
provides developers with information on how to operate the reference implementation of the Secure
Device Onboard protocol.

[Protocol Specification](protocol-specification/introduction.md) documentation provides developers
with complete details of the Secure Device Onboard protocol.

## Project Repositories

Component | Source Repository
------------------------------------|----------------------------------------------------------
All-in-one Demo | <https://github.com/secure-device-onboard/all-in-one-demo>
Client SDK | <https://github.com/secure-device-onboard/client-sdk>
Reseller and Manufacturing Toolkits | <https://github.com/secure-device-onboard/supply-chain-tools>
IOT Platform SDK | <https://github.com/secure-device-onboard/iot-platform-sdk>
Rendezvous Service | <https://github.com/secure-device-onboard/rendezvous-service>
Protocol Reference Implementation | <https://github.com/secure-device-onboard/pri>

## Terminology
 

| Term                                                | Description |
|-----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| API                                                 | Acronym - Application Programming Interface. An Application Programming Interface is a set of subroutine definitions, protocols, and tools for building application software. |
| Board or System OEM                                 | Term - An IoT Device manufacturer (OEM/ODM) who designs the hardware & software, assembles a board and/or system using a processor.|
| CA                                                  | Acronym - Certificate Authority. In cryptography, a certificate authority or certification authority (CA) is an entity that issues digital certificates. A digital certificate certifies the ownership of a public key by the named subject of the certificate.|
| Client                                              | Term – Secure Device Onboard code that implements the Secure Device Onboard Ownership transfer within an IoT Device.|
| CoAP                                                | Acronym - Constrained Application Protocol. A specialized web transfer protocol for use with constrained nodes and constrained networks in the Internet of Things. The protocol is designed for machine-to-machine (M2M) applications such as smart energy and building automation.|
| COS                                                 | Acronym - Code Origination Scan. An audit process for software code. Run on created software that may be using open source code or samples. It allows companies to properly attribute all code snippets and usage to the correct owners as well as any hazardous code usage. The code scan is a discovery process that removes uncertainties around IP ownership.|
| Counter-signed Ownership Voucher                    | Term - The Ownership Voucher can be extended by each entity in the supply chain that legally and physically possesses/owns the IoT Device on the way to its ultimate Owner. The counter-signing is accomplished electronically by adding a subsequent signature to the (counter-signed) Ownership Voucher received from the previous possessor in the Supply Chain – without having to open the shipping container and without powering the IoT Device on.|
| CRL                                                 | Certificate Revocation List|
| CSP                                                 | Acronym - Cloud Service Provider. A cloud provider is a company that offers some component of cloud computing – typically Infrastructure as a Service (IaaS), Software as a Service (SaaS) or Platform as a Service (PaaS) – to other businesses or individuals.|
| CSR                                                 | Acronym - Certificate Signing Request|
| DAA                                                 | Acronym - Direct Anonymous Attestation.|
| Device                                              | A gateway, microprocessor, or microcontroller that is capable of sending information via a network protocol such as HTTP or Bluetooth® and is enabled with Secure Device Onboard.|
| Device Credentials                                  | Term - The Ownership data embedded in a Secure Device Onboard enabled-device used to establish Trust with an IOT Platform Service by proving the authenticity of the Ownership Voucher presented by the Secure Device Onboard Owner.|
| Device GUID                                         | Term/Acronym - Device Global Unique Identification. A Globally Unique ID that is a not-easily guessable ID that is generally randomly generated and sparsely populated within a large set of possible values. It is embedded in the IoT Device, included in the Ownership Voucher and printed on the outside of the shipping container. It is used to extend the Ownership Voucher without opening the shipping container and to identify an IoT Device within the Secure Device Onboard service as an index for the rendezvous between the IoT Platform Service and the IoT Device.|
| Device Management Agent                             | Term - Code that runs on an IoT Device to provide the normal, day-to-day, minute-to-minute interaction with the Device Management Service after the IoT Device is onboarded. The agent can be distributed with the Secure Device Onboard device, or downloaded to the Device during Secure Device Onboard onboarding.|
| Device Management Service                           | Term - Back end service used by customer that ultimately will take ownership-on-behalf-of an IoT Device to manage services, security or access. Can include Cloud Service Provider (CSP), Identity and Access Management platform (IAM), IoT Platform Management services (like Building Management Services (BMS), Oil and Gas platform management services, etc.) and other Independent Software Vendors (ISV). The Secure Device Onboard Owner functions are implemented within the Device Management Service to accomplish the Secure Device Onboard ownership transfer on behalf of the service in the legal owner’s account.|
| DHKEX                                               | Acronym - Diffie-Hellman Key EXchange protocol. A method of securely exchanging cryptographic keys over a public channel and was one of the first public-key protocols as originally conceptualized by Ralph Merkle and named after Whitfield Diffie and Martin Hellman. The protocol is one of the earliest practical examples of public key exchange that is implemented within the field of cryptography. |
| DI                                                  | Device Initialization |
| Distributor/Reseller                                | Term - An entity in the supply chain between the Board/System OEM and the Owner. |
| DSA                                                 | Acronym - Digital Signature Algorithm. The Digital Signature Algorithm (DSA) is a Federal Information Processing Standard for digital signatures. |
| ECC                                                 | Acronym - Elliptical Curve Cryptography. Elliptic Curve Cryptography Elliptic curve cryptography is an approach to public key cryptography based on the algebraic structure of elliptic curves over finite fields. |
| EC-DSA                                              | Acronym - Elliptic Curve DSA. The Elliptic Curve Digital Signature Algorithm (ECDSA) is a variant of the Digital Signature Algorithm (DSA) which uses elliptic curve cryptography. |
| EPID (group) public-key, EPID (private) member-key  | Term - EPID keys come as Group keys; there is one public-key associated with many private-keys in a group. All members of a group can sign with their unique member-key and the signature can be verified with the group public-key, without revealing which specific member did the signing (privacy attribute). |
| Group based revocation                              | Term - The issuer revokes the whole Intel<sup>®</sup> EPID group by revoking the group public key. Group based revocation is expected to be a rare event and would only happen under limited criteria. The GroupRL nomenclature is used to denote the revocation list corresponding to this method. |
| GroupRL                                             | Acronym - Group Revocation List. A list of certificates that have been revoked and are no longer valid. The GroupRL nomenclature is used to denote the revocation list corresponding to this method. |
| GUID                                                | Acronym - Globally Unique Identifier. A GUID is a term used for a number that is generated to create a unique identity for an entity such as a document. |
| Hardware Root of Trust                              | Term - Acronym - HWRoT - A hardware key provisioned into the device during its manufacture, whose signatures can be verified to have this providence. |
| HMAC                                                | Acronym - Hash-based Message Authentication Code. In cryptography, a keyed-hash message authentication code (HMAC) is a specific type of message authentication code (MAC) involving a cryptographic hash function and a secret cryptographic key. |
| IANA                                                | Acronym - Internet Assigned Numbers Authority. The global coordination of the DNS Root, IP addressing, and other Internet protocol resources is performed as the Internet Assigned Numbers Authority. |
| Installer Tool                                      | Term – Android based tool to provide network connection to Secure Device Onboard Clients |
| Device OEM/ODM                                      | Term - A Board or System OEM/ODM that builds an IoT Device. |
| Intel Client                                        | Term – Secure Device Onboard Client implementation done on an Intel X86-based platform using Intel<sup>®</sup> ME technology. |
| Intel<sup>®</sup> EPID                              | Acronym - Intel<sup>®</sup> Enhanced Privacy ID. Intel<sup>®</sup> EPID is an algorithm developed by Intel Corporation for attestation of a trusted system while preserving privacy. |
| Intel<sup>®</sup> ME                                | Acronym – Intel<sup>®</sup> Management Engine. A secondary (service) processor located on the motherboard, and uses TLS-secured communication and strong encryption to provide additional security. |
| IoT Attack Map                                      | Term - A map of devices and the infrastructure they inhabit, control or monitor, that allows an attacker to plan or implement an attack on those devices or that physical infrastructure because of a cyberspace vulnerability that exists now or is discovered the future. |
| IoT Device                                          | Term - A “Thing” on the Internet of Things. An embedded system that usually resides at or near the edge of a network and provides functionality more in a Machine-to-Machine (M2M) fashion than with a direct user-interface. |
| IOT or IoT                                          | Acronym - Internet of Things |
| IoT Platform                                        | Term – Software solution to manage IoT devices. |
| IoT Platform SDK                                    | Term – Secure Device Onboard code that implements the Secure Device Onboard Ownership transfer on behalf of the IoT Platform Service. Usually integrated with or contained in the IOT Platform or Device Management Service. |
| JHI                                                 | Acronym – Java* Host Interface |
| JKS                                                 | Acronym - Java* Keystore File.A file with extension jks serves as keystore. The Java Development Kit maintains a CA keystore in folder jre/lib/security/cacerts. JDKs provide a tool named keytool to manipulate the keystore. |
| JNI                                                 | Acronym – Java* Native Interface. A programming framework that enables Java code running in a Java Virtual Machine (JVM) to call and be called by native applications (programs specific to a hardware and operating system platform) and libraries written in other languages such as C, C++ and assembly. |
| JSON*                                                | Acronym - JavaScript Object Notation. A lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate. |
| KDC                                                 | Acronym - Key Distribution Center. Acts as both an Authentication Server and as a Ticket Granting Server. When a client needs to access a resource on the server, the user credentials (password, Smart Card, biometrics) are presented to the Key Distribution Center for authentication. If the user credentials are successfully verified in the Key Distribution Center then a Ticket Granting Ticket (TGT) is issued to the client. The TGT is cached in the local machine for future use. The TGT expires when the user disconnects or log off the network, or after it expires. The default expiry time is one day (86400 seconds). |
| KDF                                                 | Acronym - Key Derivation Function. A key derivation function (KDF) derives one or more secret keys from a secret value such as a master key, a password, or a passphrase using a pseudo-random function. |
| Manufacturer                                        | The initial installer of Secure Device Onboard onto a device. |
| Manufacturing Credential                            | Term - An ownership credential stored in the device during manufacturing that contains the identity of the manufacturer and a device info string that indicates the kind of device this is. It does not otherwise identify the Device. |
| MCU                                                 | Acronym - Microcontroller Unit. A small computer on a single integrated circuit. |
| Modules                                             | Modules are defined by the device manufacturer (ODM) and are pieces of code that have a specific name and perform a specific function. Firmware update, key provisioning, and Wi-Fi* network setup are some examples of common functionality that could be provided by modules. |
| NO                                                  | Acronym – New Owner identity protocol |
| OC                                                  | Ownership Credentials |
| OCF                                                 | Acronym - Open Conductivity Framework or Open Conductivity Forum |
| OCSP                                                | Online Certificate Status Protocol |
| ODM                                                 | Acronym - An Original Design Manufacturer (ODM) is a company that designs and manufactures a product as specified and eventually rebranded by another firm for sale. |
| Onboarding                                          | Term - Establishing a two-way trust relationship between an IoT Device and a Device Management Service, including provisioning of credentials and instructions/policies to allow the IoT Device to be controlled by the Device Management Service. |
| OP                                                  | Older acronym for Ownership Voucher |
| OPS                                                 | Acronym – Owner Protocol Service |
| OSI                                                 | Acronym – Owner Service Information. Owner Service Information is information accepted by the module, from the Owner Service. |
| OV                                                  | Ownership Voucher |
| Owner                                               | Same as ‘IoT Platform SDK’ |
| Owner                                               | Term – The final purchaser and legal owner of an IoT Device. |
| Ownership Credential                                | Term - In Secure Device Onboard, a credential stored in the Device during manufacturing that contains a public key and a GUID. |
| Ownership Voucher                                   | Term - An electronic ownership record used to prove ownership to the IoT Device. It can be extended in the supply chain by adding subsequent signatures. The Owner provides the Ownership Voucher to the IoT Platform of their choosing. The Device Management Service uses the Ownership Voucher to register its ownership-on-behalf-of claim with the Rendezvous Service and then prove it to the IoT Device. |
| PEM                                                 | Acronym – Privacy-Enhanced Mail. An Internet standard that provides for secure exchange of electronic mail. |
| PKI                                                 | Acronym - Public Key Infrastructure. A set of roles, policies, and procedures needed to create, manage, distribute, use, store, and revoke digital certificates and manage public-key encryption |
| PRI                                                 | Acronym - Protocol Reference Implementation. A reference implementation of the Secure Device Onboard protocols. |
| Private-key based revocation                        | Term - The issuer revokes a member based on the member’s private key. This revocation method is used when a member’s private key becomes exposed and subject to revocation criteria. The PrivRL nomenclature is used to denote the revocation list corresponding to the private key list. |
| PrivRL                                              | Acronym - Private-key based Revocation List. See above. |
| PSI                                                 | Acronym – Pre-Service Info. Pre-Service Info consists of an Owner Server’s expectations in terms of module support. |
| PXE                                                 | Acronym - Preboot eXecution Environment. A specification describes a standardized client-server environment that boots a software assembly, retrieved from a network, on PXE-enabled clients. On the client-side, it requires only a PXE-capable network interface controller (NIC), and uses a small set of industry-standard network protocols such as DHCP and TFTP. |
| Rendezvous Service                                  | Term - A service to arrange a rendezvous between two entities that want to establish trust between them. Secure Device Onboard does not create the trust, rather, after the two devices rendezvous, they independently establish Trust between themselves. |
| Retailer                                            | Term - An entity at the final transaction in the supply chain that sells an IoT Device directly to an Owner. |
| RNG                                                 | Acronym - Random Number Generator. The generation of a sequence of numbers or symbols that cannot be reasonably predicted better than by a random chance. |
| RoT                                                 | Acronym - Root of Trust. An authoritative entity for which trust is assumed and not derived. |
| RTOS                                                | Real-Time Operating System |
| RV                                                  | Rendezvous Service |
| SDK                                                 | Acronym - Software Development ToolKit |
| Secure Device Onboard                               | Secure Device Onboard is the name of the protocol used for onboarding the devices to IoT Platform. |
| Secure Device Onboard IoT Device Agent              | Term - The application that runs at initial boot on an IoT Device to accomplish “zero touch” onboarding with the Device Management Service using the Rendezvous Service. |
| Secure Device Onboard Protocols                     | Term - The API code used by the Device Management Service to enable their services. (For example, take in the Ownership Voucher, associate it with a registered user account and interact with the Rendezvous Service and the IoT Device Secure Device Onboard Onboarding Agent. |
| Secure Device Onboard URL                           | Acronym – The Secure Device Onboard URL that is embedded in an IoT device used to contact the Rendezvous Service upon first time power-on to accomplish the rendezvous protocol with its device management service. |
| SEK                                                 | Session Encryption Key |
| SHA | Acronym - Secure Hash Algorithm |
| Signature based revocation                          | Term - The issuer revokes a member based on the signature generated by the member. This revocation method is used when a key becomes subject to revocation criteria but has not be exposed yet. The SigRL nomenclature is used to denote the revocation list corresponding to this method. |
| SigRL                                               | Acronym - Signature based Revocation List. See above. |
| Supply Chain Toolkit                                | Term – Secure Device Onboard code that performs device initialization, management of signing keys, and counter-signing of ownership vouchers while a device is travelling through the supply chain. |
| SVK                                                 | Session Verification Key |
| TA                                                  | Acronym - Trusted Application. Trusted applications are applications that are signed to run within a TEE. |
| TB                                                  | Acronym - Trust Broker. A specifically identified agent that can manage trust between entities. |
| TCB                                                 | Acronym - Trusted Computing Base. A computer system is the set of all hardware, firmware, and/or software components that are critical to its security, in the sense that bugs or vulnerabilities occurring inside the TCB might jeopardize the security properties of the entire system. |
| TEE                                                 | Acronym - Trusted Execution Environment. A segmented computing environment which runs a body of code that is believed to be trusted when installed. The reason for this original belief can vary. It is typical to cryptographically link a TEE back to its original trusted state, such as using digital signatures or crypto-hashes to prove that it has not been tampered with. The key used to do this is commonly called the Hardware Root of Trust. |
| TLS                                                 | Acronym - Transport Level Security |
| TO0                                                 | Acronym - Transfer of Ownership, step 0: establishes a connection with the Owner to the Rendezvous server and creates trust that it has ownership of an Ownership Voucher so Service Info can be transmitted. |
| TO0Client                                           | A utility program that runs the TO0 protocol. It takes an Ownership Voucher as input, runs the TO0 protocol, resulting in the OP being stored in the Rendezvous server. |
| TO1                                                 | Acronym - Transfer of Ownership, step 1: Establishes a connection from a Device to the Rendezvous server. |
| TO2                                                 | Acronym - Transfer of Ownership, step 2: Establishes a trusted, secure connection between the Device and the final Owner and enables the Owner to configure the device as per the requirement from DMS. |
| TOFU                                                | Acronym - Trust On First Use |
| TPM                                                 | Acronym - Trusted Platform Module |
| URI                                                 | Uniform Resource Identifier (URI) |
| URL                                                 | Acronym - Uniform Resource Locator. A web address, is a reference to a web resource that specifies its location on a computer network and a mechanism for retrieving it. A URL is a specific type of Uniform Resource Identifier (URI). |
| Verifier Exclusion List Revocation                       | When the Verifier suspects that a member key has been compromised based on the verifiers key revocation criteria, the verifier will add the member to an exclusion list based on the name-based signature generated by the member. The VerifierRL nomenclature is used to denote the revocation list corresponding to this method. |
| VerifierRL                                          | Acronym - Verifier Revocation List. See above. |
