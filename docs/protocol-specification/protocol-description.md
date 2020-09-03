# Protocol Description

Secure Device Onboard protocols pass JavaScript\* Object
Notation (JSON)-based messages between cooperating entities, which are listed in
subsequent sections. The messages are defined independent of any transport
protocol, permitting Secure Device Onboard to operate over multiple transport protocols with
different properties, such as:

-   RESTful HTTP/HTTPS (Current implementation of Secure Device Onboard)

-   Constrained Application Protocol (CoAP)
    [[RFC7252]]

-   TCP or TCP/TLS streams

-   Non-Internet protocols, such as Bluetooth<sup>®</sup> specification or USB\*
    specification

Secure Device Onboard messages are encoded using a distinguished encoding of JSON\*, and are
intended to fit in the RESTful ecosystem. The encoding is read-compatible with
JSON\*, so that JSON\* decoders and JSON\* pretty-printers will
work well with Secure Device Onboard messages. However, standard JSON\* components may need to have their output
adjusted to conform to this encoding. Signatures refer to the transmitted form
of a message, so a standard JSON\* parser may have reason to keep the original
JSON\* text around for verifying signatures.

Other encodings of JSON* exist, and might be used with Secure Device Onboard in the future,
such as the Concise Binary Object Representation (CBOR) encoding described in
[[RFC7049]].

## Message Passing Protocol

Secure Device Onboard messages are defined in  section [§](../data-transmission-persistence/#data-transmission-persistence). A
message is logically encapsulated by a protocol-dependent header containing the
message type, protocol version, and other transmission-dependent
characteristics, such as the message URL and message length in bytes. The
message header is transmitted differently for different transport protocols. For
example, the message header may be encoded into the HTTP header fields for
RESTful protocols.

The message body is a JSON\* object, encoded within the JSON\* distinguished
encoding, described in section [§](#json-distinguished-encoding) : JSON* Distinguished Encoding.

## JSON* Distinguished Encoding

The JSON\* specification only describes a textual form of JSON\*. Any time JSON\* is
transmitted, it must be encoded into a particular character set, bit order, and others.
Secure Device Onboard uses a restrictive encoding for JSON\* to make it easier for a
constrained device to quickly and efficiently parse messages. We believe that
this restricted encoding permits parsers with similar code size to parsing
binary message formats. Since the encoding preserves many of the JSON\* textual
properties, standard JSON\* components may also be used when available (for example, in
non-constrained environments). However, the output of these components must be
re-encoded before messages are transmitted to ensure compatibility with the
distinguished encoding.

The following table describes how JSON* is encoded into the JSON\* distinguished
encoding.

<table>
    <caption>Table 1 - JSON* Distinguished Encoding</caption>
    <thead>
        <tr>
            <th>Encoding Rule</th>
            <th>Motivation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Each message is a valid JSON* object</td>
            <td>Secure Device Onboard ecosystem can interpret message</td>
        </tr>
        <tr>
            <td>Messages are encoded only in printable ASCII (codes 0x20-0x7e). Non-ASCII characters in strings are encoded as Unicode: &bsol;uXXXX (or &bsol;uXXXX&bsol;uYYYY for 32-bit Unicode)</td>
            <td>Simplicity of interpreter.</td>
        </tr>
        <tr>
            <td>Special characters: {}[]&amp;"&bsol; must be encoded as Unicode &bsol;u… sequences when they appear in strings. Backslash escaping is not used.</td>
            <td>Simplicity of interpreter.</td>
        </tr>
        <tr>
            <td>Objects are stored in a distinguished order, as defined in this document. The object elements may not be reordered.</td>
            <td>Permits receiver to verify signatures without needing to sort the message’ object tags; receiver can check expected object tags rather than parsing.</td>
        </tr>
        <tr>
            <td>No whitespace, newlines, carriage returns or tabs are permitted as padding. Spaces and tabs are permitted in JSON strings where not otherwise restricted.</td>
            <td>Simplicity of interpreter, verification of signatures.</td>
        </tr>
        <tr>
            <td>No JSON* comments are permitted.</td>
            <td>Simplifies verification of signatures.</td>
        </tr>
        <tr>
            <td>Object tags are chosen for brevity</td>
            <td>Decreases message size, important for constrained systems. When the same value appears multiple times in the Secure Device Onboard protocols, we use an object tag containing the object type and an integer giving the instance the object is used within this specification. For example, Nonces are tagged as “n1”, “n2”, and so on.</td>
        </tr>
        <tr>
            <td>Byte arrays (ByteArray) are stored as strings containing base64 data; the message structure gives a length field preceding the string to make parsing memory efficient.</td>
            <td>Decreases message size.</td>
        </tr>
        <tr>
            <td>Only positive integers are encoded; each integer is encoded in the shortest possible decimal representation (For example, “01” or “0x33” are not permitted). Negative integers and floating point numbers are not encoded.</td>
            <td>Simplicity of parser, compactness of representation, ability to sign messages.</td>
        </tr>
        <tr>
            <td>True is encoded as the number 1 False is encoded as the number 0</td>
            <td>Simplicity of parser</td>
        </tr>
    </tbody>
</table>

When messages are signed, hashed, or HMAC’d the distinguished encoding of the
message is used. Since this coding can only have one form for any message value
(i.e., it is distinguished), this can be done directly on the encoded text.
Conventional JSON* decoders might find it convenient to keep pointers into the
original message text of a message for these operations, since their operation
might introduce or remove parts of the plaintext.


!!! Note 
    The message and data type descriptions in this document use JSON with
    comments and white space. This is for clarity only. JSON comments and white
    space are never transmitted.


In protocol implementations, JSON\* is usually parsed into a parse tree by a
general purpose parser, then the protocol implementation reads the parsed data
to verify all message components are there. Because of the distinguished
encoding, JSON\* messages can be parsed by code that is intended to recognize the
expected message, data type, or object tag. For example, code to parse an object:

```json
{"s1":"hello","n":25,"b":[4,"KlMyRg=="]}
```

can recognize the object using straight line code as follows:

1.  Allocate variables s1, n, b[], bsize

2.  Verify begin object {

3.  Verify tag “s1”, then a string, then a comma

4.  Let s1 = string that is read

5.  Verify tag “n”, then a number, then a comma

6.  Let n = number that is read

7.  Verify tag “b” then a sequence.

8.  Let bsize = the first element of the sequence, then decode the base64 string
    into an array of bsize bytes (yielding 0x2a, 0x53, 0x2a, 0x66).

9.  Verify end object }

Although this seems laborious on paper, it results in very small and efficient
code that can be debugged easily.

In addition, the distinguished subset removes the need for complex object
escaping, such as embedding a double quote in a string ("a &bsol;"b&bsol;" c" must be
encoded as: "a &bsol;u0022b&bsol;u0022 c"). **POST** body encoding and decoding is also
avoided, as well as UTF escaping. This burden is placed instead on the client
protocol. However, we do not believe this is a high burden for a Device
implementation.

## Protocol Entities

See Figure ‎1. Secure Device Onboard Entities and Entity
Interconnection for a diagram of Secure Device Onboard Entities and their protocol
interconnections.

-   **Manufacturer (Mfg)**: This is an Secure Device Onboard application running in the
    factory, which implements the initial communications with the Device TEE, as
    part of the Device Initialize Protocol (DI).

-   **Device**: The device being manufactured, later the device being
    provisioned. This device has hardware and software configured on it,
    including a Device TEE and a Device to Manager Agent. In the following
    documentation, an Secure Device Onboard enabled Device is capitalized.

    -   **Device TEE**: The Trusted Execution Environment within the Device. In some
    Devices, this is a co-processor [For example Intel<sup>®</sup> Management Engine (Intel<sup>®</sup> ME)]
    or a special processor mode [For example, Intel<sup>®</sup> Software Guard Extensions (Intel<sup>®</sup>
    SGX)] that enables a small kernel of code to run, with credentials to prove
    its authenticity. Many Intel devices implement an Intel<sup>®</sup> DAL to allow new
    (signed) applications to be added to the Management Engine.

    -   **Device TEE App**: This is the application that is installed in the TEE of
    the device to provide the Secure Device Onboard capabilities on the device. When we
    informally refer to the Device TEE as an endpoint to a protocol, we always
    mean the Device TEE App.

    -   **Device to Manager Agent**: Software that runs on the device in normal
    operation that connects the device to its manager across the network. This
    entity’s function is specific to the Manager, and outside the scope of this
    document, except for its first connection to the Manager. Our intention is
    that the Device to Manager Agent matches as closely as possible the existing
    agents that connection devices to remote network or cloud managers.

-   **Owner**: This is an entity that is able to prove ownership to the
    **Device** using an Ownership Voucher and a private key for the last entry
    of the Ownership Voucher (the “Owner Key”). Various members of the supply
    chain may have bought and sold the device while it was still “boxed,” acting
    as owners, but without powering on the device. The final owner in the chain
    uses the Owner Client to provision the device, and then controls it across a
    network using a Manager.

    -   **Manager**: The entity that manages devices across a network. This can
    range from an application on a user’s computer, phone or tablet, to an
    enterprise server, to a cloud service spanning multiple geographic regions.
    The Manager interacts with the device using the **Device** to **Manager
    Agent**. Commonly, the Manager is an existing management system or cloud
    management service that is provisioned using Secure Device Onboard, so that it operates
    the same as if it were manually provisioned.

    -   In some cases, the owner elects to subscribe to a cloud service and proxy
    his ownership, so that the Manager controls the ownership credentials of the
    owner. We believe this to be a growing trend for IoT devices.

    -   **Owner Client**: This is an entity constructed to perform Secure Device Onboard
    protocols on behalf of the Owner. The Owner Client is an application that
    executes on some platform already controlled by the owner. After the
    protocols are completed, the Owner Client transfers control of the device to
    the Owner’s Manager, and never interacts with the device again.

-   **Rendezvous Server**: A service on the Internet that acts as a rendezvous
    point between a newly powered on Device and the Owner Client.

-   **Management Service**: The entity that uses the Secure Device Onboard Owner Client to
    take ownership of the Device, so that it can manage the device remotely
    using its own management techniques (protocols, and others). During Secure Device Onboard
    operation, the Management Service interacts with the Management Agent via
    the ServiceInfo (section [§](../detailed-protocol-description/#pmserviceinfo-type-5)) key-value pairs.


-   **Management Agent**: The entity that uses the Secure Device Onboard Device software to
    allow the device ownership to be transferred using Secure Device Onboard protocols.
    During Secure Device Onboard operation, the Management Agent interacts with the
    Management Service via the ServiceInfo key-value pairs.

### Entity Credentials 

Each of the entities above identifies itself in Secure Device Onboard protocols using
cryptographic credentials. These are:

-   **Device Attestation Key** : Secure Device Onboard uses cryptographic device
    attestation. The protocol can support many mechanisms for device attestation
    but this spec supports two basic capabilities: Intel<sup>®</sup> EPID and ECDSA. For
    each of the methods, there is a private key that is provisioned into the
    device, such as when the CPU is manufactured (chip manufacture time) for
    establishing the trust for a Trusted Execution Environment (TEE) that runs
    on the device. Applications in the TEE are identified by an application
    identifier. When signed by the device attestation key, this provides
    evidence of the code being executed in the TEE.

-   **Ownership Credential Key Pair:** This is a key pair that serves
    temporarily to identify the current owner of the device. When the device is
    manufactured, the manufacturer uses a key pair to put in an initial
    ownership credential. Later, the protocols shall conspire specifically to
    replace this credential with a new ownership credential, effecting ownership
    transfer.

-   The **Ownership Credential** does not identify the owner in general, it
    identifies the owner for the purposes of ownership transfer. The
    manufacturer’s ownership credential, as stored in the device, must match the
    credential at one side of the ownership voucher. That is all. It is not
    intended that this key pair permanently identify the manufacturer or any of
    the parties in the ownership voucher. On the contrary, we expect that the
    manufacturer will use different keys over time and the owners will also use
    different keys over time, specifically to obscure their identity in the
    Secure Device Onboard protocols and increase of the robustness of Secure Device Onboard.

### Management Agent/Service interactions using ServiceInfo

In the Transfer Ownership Protocol 2 (TO2), after mutual trust is proven, and a
secure channel is established, key-value pairs are exchanged. This is a
mechanism for interaction between the Management Agent and Management Service
using the TO2 protocol as a secure transport. The amount of information
transferred using this mechanism is not specifically constrained by the TO2
protocol, but some structure is imposed in the definition of ServiceInfo
(Section [§](../detailed-protocol-description/#pmserviceinfo-type-5)‎). The intent is to allow the Management Service to provision
sufficient keys, data and executables to the Management Agent so that they are
enabled to interact securely for the life of the device.

For example, a Management Agent may send a Public Key Cryptography Standards
(PKCS\#10) Certificate Signing Request (CSR) to the Management Service in a
Device ServiceInfo key-value pair, which can use a certificate authority (CA) to
provision a certificate trusted by itself and send that certificate back to the
Management Agent in PKCS\#7 format, using an Owner ServiceInfo key-value pair.

The flows of ServiceInfo information between the Owner and the Management
Service, and between the Device and the Management Agent, are outside the scope
of this document.

ServiceInfo provides a key-value pair mechanism. The namespace of keys is
divided into module-specific spaces and key attributes allow for downloading of
data files or executable code (For example, installation scripts) using the trust
provided by Secure Device Onboard.

## Protocol Entity Interactions

The following diagram shows the interaction between the protocol entities in the
Secure Device Onboard Protocols:

<figure id="fig-EntitiesEntity">
<!--  <img src="../img/4cae3f07e3037cb0969aa57b0bec67c9.png" /> -->
  <img src="../img/fig-agent-interaction.png" alt="entity interactions">
  <figcaption>Figure 1 - Secure Device Onboard Entities and Entity Interconnection</figcaption>
</figure>

The following sections define these protocols.

It is expected the “final state” protocol (bottom arrow in the diagram) may be a
pre-existing protocol between a manager agent and manager service that exist
independently of Secure Device Onboard. Secure Device Onboard serves, then, to provide credentials
rapidly and securely so that the pre-existing software is able to take over and
operate, as if it were manually configured. Secure Device Onboard is then not used by the
device or owner unless the owner wishes to re-provision the device, such as to
effect another ownership transfer.

Some of the interactions between entities are not defined in the protocols:

-   The **manufacturer** creates an Ownership Voucher based on the credentials
    in the Device Initialize Protocol (DI). The Ownership Voucher is a digital
    document that provides the Owner with the credentials to take ownership of
    the Device. It is extended with each owner while the device is offline
    (that is, boxed or shipped) between Manufacturer and Owner. The Ownership
    Voucher is defined in section [§](#the-ownership-voucher). This specification does not indicate
    how the Ownership Voucher is transported from the Manufacturer to the Owner
    Client, where it is used in the Secure Device Onboard protocols.

-   The interaction between the **Device TEE App** and the **Device** to
    **Manager Agent** is system dependent.

-   The interaction between the **Owner’s Manager Service** and the **Owner
    Client** is dependent on the implementation of these two components.

## Protocols

The following protocols are defined as part of Secure Device Onboard. Each protocol is
identified with an abbreviation, suitable to use as a programming prefix. The
abbreviations are also used in this discussion.

<table>
    <caption>Table 2 - Secure Device Onboard Protocols</caption>
    <thead>
        <tr>
            <th>Protocol Name</th>
            <th>Abbr.</th>
            <th>Function</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Device Initialize Protocol (DI)</td>
            <td>DI</td>
            <td>For insertion of Secure Device Onboard credentials into device during the manufacturing process.</td>
        </tr>
        <tr>
            <td>Transfer Ownership Protocol 0 (TO0)</td>
            <td>TO0</td>
            <td>Secure Device Onboard Owner identifies itself to Rendezvous Server. Establishes the mapping of GUID to the Owner IP address.</td>
        </tr>
        <tr>
            <td>Transfer Ownership Protocol 1 (TO1)</td>
            <td>TO1</td>
            <td>Device identifies itself to the Rendezvous Server. Obtains mapping to connect to the Owner’s IP address.</td>
        </tr>
        <tr>
            <td>Transfer Ownership Protocol 2 (TO2)</td>
            <td>TO2</td>
            <td>Device contacts Owner. Establishes trust and then performs Ownership Transfer.</td>
        </tr>
    </tbody>
</table>

The following figure shows a graphical overview of these protocols. Graphical
representations of each protocol are presented with the protocol details.

<figure id="fig-protocols">
<!--    <img src="../img/78ea5633164d5e07e801a49207f5476b.emf" alt="Secure Device Onboard Protocols"/> -->
    <img src="../img/fig-protocols-bounce.png" alt="Secure Device Onboard Protocols">
    <figcaption>Figure 2 - Graphical Representation of the Secure Device Onboard Protocols</figcaption>
</figure>

### Device Initialize Protocol (DI)

The Device Initialize Protocol (DI) runs within the factory when a new device is
completed. The protocol’s function is to embed the ownership and manufacturing
credentials into the newly created device’s TEE. This prepares the device and
establishes the first in a chain for creating an Ownership Voucher with which to
transfer ownership of the device.

The Device Initialize Protocol assumes that the protocol will be run in a safe
environment. The trust model is Trust on First Use (TOFU). When possible, the DI
Protocol should use write-once memory to ensure the Device is not erased or
reprogrammed after factory use. When no such hardware is available, it might be
possible to reprogram the device, so as to create alternate Secure Device Onboard
credentials.

The **Device Initialize Protocol** starts with:

-   The physical device and the **Secure Device Onboard Manufacturing Component** attached
    to a local network within the factory.

-   The **Secure Device Onboard Manufacturing Component** has access to:

    -   A **key pair** for device ownership, which will be used to create ownership
    credentials in the device and the Ownership Voucher. This key pair does not
    specifically identify the manufacturer (For example, it is not in a certificate)
    and may be changed from time to time, so long as the ownership credentials
    refer to the same key pair as the Ownership Voucher for that device.

    -   Certificate for the device manufacturer, which is published at the
    Rendezvous Service URL.

    -   Device description string, configured by the manufacturer.

-   **Device TEE** running the Secure Device Onboard application.

The **Device Initialize Protocol** ends with:

-   The **Secure Device Onboard Manufacturing Component** has information and credentials to
    create an Ownership Voucher for the device or has the Ownership Voucher
    itself.

-   The **Device** has ownership and manufacturer credentials stored in its TEE.
    The **Device** should arrange to protect these credentials. Ideally:

    -   Only the **Device TEE** software should be able to access these credentials.

    -   The credentials are protected against modification by non-Secure Device Onboard
    programs.

    -   Any modification of the credentials by non-Secure Device Onboard programs (despite
    measures above) is detectable.

-   The **Device** is ready to be powered off and boxed for shipment. No further
    network attachment is necessary.

-   The **Device** has a GUID that can be used to identify it to its new owner.
    This GUID is also known to the Secure Device Onboard Manufacturing Component. The GUID
    is not a secret. Specifically, the GUID is intended to be visible to the
    Owner when the device shipped in a box, perhaps being on the box itself with
    a bar code, perhaps being on the bill of lading. The GUID is used for one
    Secure Device Onboard transfer of ownership only; after Transfer Ownership Protocol 2,
    the GUID is replaced, and the Device has no memory of the original GUID.

### Transfer Ownership Protocol 0 (TO0)

Transfer Ownership Protocol 0 (TO0) serves to connect the Owner Client with the
Rendezvous Server. In this protocol, the Owner Client indicates its intention
and proves it is capable of taking control of a specific Device, based on the
Device’s current GUID.

**Transfer Ownership Protocol 0** starts with:

-   A **Device** that has undergone the Device Initialize Protocol (DI) and thus
    has credentials in its TEE identifying the Manufacturer public key that is
    in the Ownership Voucher.

-   The **Owner Client** has access to the following:

    -   An **Ownership Voucher**, whose last Public key belongs to the Owner, and
    the GUID of the device, which is also authorized by the Ownership Voucher.

    -   The **private key** that is associated with its public key in the Ownership
    Voucher.

    -   An **IP address** from which to operate. This IP address need bear no
    relationship to the service addresses that are used by the Owner. The Owner
    may take steps to hide its address, such as allocating it dynamically (For example,
    using DHCP) or using an IPv6 privacy address. The motivation for hiding this
    IP address is to maintain the privacy of the Owner from the Rendezvous
    Server or from anyone monitoring network traffic in the vicinity of the
    Rendezvous Server. This can never be done for sure; we think of it as
    raising the bar on an attacker.

-   The **Rendezvous Server** has some way to trust at least one key in the
    Ownership Voucher. For example, the Manufacturer has selected the Rendezvous
    Server, then the Rendezvous Server might be aware of the Manufacturer’s
    public key used in the Ownership Voucher.

**Transfer Ownership Protocol 0** ends with:

-   The **Rendezvous Server** has an entry in a table that associates the Device
    GUID with the Owner Client’s DNS name and/or IP address for some fixed
    amount of time.

-   The **Owner Client** is waiting for a connection from the Device TEE at this
    DNS name and/or IP address for this same amount of time.

If the Device TEE appears within the set time interval, it can complete Transfer
Ownership Protocol 1 (TO1). Otherwise, the Rendezvous Server forgets the
relationship between GUID, IP address, and the Owner Client must perform
Transfer Ownership Protocol 0 again.

In the case of a Device being connected to a cloud service, the Owner Client
typically would repeatedly perform the TO0 Protocol until all devices known to
it successfully complete the TO0 Protocol. In the case of a Device being
connected using an application program implementation of the Owner Client, the
Owner might arrange to turn on the Owner Client shortly before turning on the
device, to expedite the protocol.

The Rendezvous Server is only trusted to faithfully remember the GUID to Owner
Secure Device Onboard Client IP/DNS mapping. The other checks performed
protect the server from DoS attacks, but are not intended to imply a greater
trust in the server. In particular, the Rendezvous Server is not trusted to
authorize device transfer of ownership. Furthermore, the Rendezvous Server never
directly learns the result of the device transfer of ownership.

### Transfer Ownership Protocol 1 (TO1)

Transfer Ownership Protocol 1 (TO1) is an interaction between the Device TEE and
the Rendezvous Server that points the Device TEE at its intended Owner Client,
which has recently completed Transfer Ownership Protocol 0. The TO1 Protocol is
thus the mirror image of the TO0 Protocol, on the Device side.

The **TO1 Protocol** starts with:

-   A **Device** that has undergone the Device Initialize Protocol (DI) and thus
    has credentials in its TEE identifying the particular Manufacturer Public
    Key that is in the Ownership Voucher.

-   An **Owner Client** and **Rendezvous Server** that have successfully
    completed Transfer Ownership Protocol 0:

-   The **Rendezvous Server** has a relationship between the GUID stored in the
    device TEE and an IP address.

-   The **Owner Client** is waiting for a connection from the Device TEE on this
    same IP address.

If these conditions are not met, the Device *will fail* to complete the TO1
Protocol. In this case, it must repeatedly try to complete the protocol with an
interval of time between tries. The interval of time should be chosen with a
random component to try to avoid congestion at the Rendezvous Server.

After the **TO1 Protocol** completes successfully:

-   The **Device** has rendezvous information sufficient to contact the Owner
    Client directly.

-   The **Owner Client** is waiting for a connection from the Device TEE on this
    same IP address (still, since it is unaffected by the TO1 Protocol).

### Transfer Ownership Protocol 2 (TO2)

Transfer Ownership Protocol 2 (TO2) is an interaction between the Device TEE and
the Owner Client where the transfer of ownership to the new Owner actually
happens.

Before the **TO2 Protocol** begins:

-   The **Owner** has received the Ownership Voucher, and run Transfer Ownership
    Protocol 0 to register its IP address against the Device GUID. It is waiting
    for a connection from the Device TEE on this same IP address.

-   The **Device** has undergone the Device Initialize Protocol (DI) and thus
    has credentials in its TEE identifying the particular Manufacturer’s Public
    Key that is (hashed) in the Ownership Voucher.

-   The **Device** has completed Transfer Ownership Protocol 1 (TO1), and thus
    has the IP address to contact the Owner Client directly.

After the **TO2 Protocol** completes successfully:

-   The **Owner Client** has replaced all the device credentials with its own,
    except for the Device’s attestation key. The Device TEE has allocated a new
    secret and given the Owner a HMAC to use in a new Ownership Voucher, which
    can be used for resale. Please see section [§](../resale-protocol/#resale-protocol) : Resale Protocol for more
    information.

-   The **Owner Client** has transferred new credentials to the Device TEE in
    the form of key-value pairs. These credentials include enough information
    for the Device TEE to invoke the correct user-mode-resident Device to
    Manager Agent and allow it to connect to the Owner’s service. The exact set
    of parameters is given in the messages: TO2.SetupDevice (section [§](../detailed-protocol-description/#to2setupdevice-type-47):
    TO2.SetupDevice, Type 47*) and TO2.OwnerServiceInfo (section [§](../detailed-protocol-description/#to2ownerserviceinfo-type-49):
    TO2.OwnerServiceInfo, Type 49*), although additional parameters may be sent
    to customize the payload.

-   The **Owner Client** has transferred these credentials to the Owner’s
    Manager, which is now ready to receive a connection from the Device.

-   The **Device TEE** has received these credentials, and has invoked the
    Device to Manager Agent and given it access to these credentials.

-   The **Device** to **Manager Agent** has received these credentials is ready
    to connect to the Owner’s Manager.

There is a distinction between: the Device TEE and the Device to Manager Agent;
and between the Owner Client and the Owner’s Manager:

-   The **Device TEE** performs the Secure Device Onboard protocols and manipulates and
    stores Secure Device Onboard credentials. The Device TEE is likely to store other
    credentials and perform other services (For example, cryptographic services) for
    the device.

-   The **Device** itself runs its basic functions in user mode. Amongst these,
    is the Device to Manager Agent, a user-mode service process that connects it
    to its remote Manager. This software is often called an “agent”, or
    “client.” We intend that this software can be a pre-existing agent for the
    Manager service chosen by the Owner.

-   The **Owner Client** is a body of software that is dedicated specifically to
    run the Secure Device Onboard Protocol on behalf of the Manager. For example, this code
    might have its own IP addresses, so that the eventual Manager IP addresses
    (which may be well known) are hidden from prying eyes.

-   The **Owner Manager** is an Internet-resident service that provides
    management services for the Owner on an ongoing basis. We intend that this
    software be a pre-existing Manager service.

After Transfer Ownership Protocol 2, the Secure Device Onboard specific software is no
longer needed until and unless a new ownership transfer is intended, such as
when the device is re-sold or if trust needs to be established anew. Secure Device Onboard
client software adjusts itself so that it does not attempt any new protocols
after the TO2 Protocol. Implementation-specific configuration can be used to
re-enable ownership transfer (For example, a CLI command).

### Key Exchange in the TO2 Protocol

Alone among Secure Device Onboard protocols, the TO2 Protocol requires message-level
encryption. The TO2 Protocol transmits potentially long-term credentials to the
Device, and these credentials are confidential between the Device TEE and its
new Owner.

The purpose of key exchange is to allow the Device and its Owner to agree on two
shared secrets. A session verification key (SVK) is used to perform a HMAC over
each message to ensure message integrity. A session encryption key (SEK) is used
to encipher each message to ensure message confidentiality.

Key Exchange starts with a protocol to construct a shared secret between the
Owner and the Device. This is accomplished using one of supported methods below,
chosen by the device. Next, the Device and Owner each uses an identical Key
Derivation Function on the shared secrets to compute the session verification
key (SVK) and the session encryption key (SEK).

The selection of a key exchange algorithm is denoted in the TO2.HelloDevice.kx
variable. When the Owner Key is RSA:

-   “**DHKEXid14**”: (Secure Device Onboard 1.0 & Secure Device Onboard 1.1 protocol spec) The
    Diffie-Hellman key exchange method using a standard Diffie-Hellman mechanism
    with a standard NIST exponent and 2048-bit modulus. This is the preferred
    method for RSA2048RESTR Owner keys.

-   **“DHKEXid15”**: The Diffie-Hellman key exchange method using a standard
    Diffie-Hellman mechanism with a standard National Institute of Standards and
    Technology (NIST) exponent and 3072-bit modulus. This is the preferred
    method for RSA 3072-bit Owner keys.

-   “**ASYMKEX**”: The Asymmetric key exchange method uses the encryption by an
    Owner key based on RSA2048RESTR; this method is useful in Secure Device Onboard Client-Intel
    environments where Diffie-Hellman computation is slow or difficult to code.

-   “**ASYMKEX3072**”: The Asymmetric key exchange method uses the encryption by
    an Owner key based on RSA with 3072-bit key.

DHKEXid14 and DHKEXid15 differ in the size of the Diffie-Hellman modulus, which
is chosen to match the RSA key size in use.

When the Owner key is ECDSA:

-   “**ECDH**”: The ECDH method uses a standard Diffie-Hellman mechanism for
    ECDSA keys. The ECC keys follow NIST P-256.

-   “ECDH384”: Standard Diffie-Hellman mechanism ECC NIST P-384 (future crypto).

The choice of key exchange algorithm follows the cryptography of the Owner key.
See section [§](#mapping-of-key-exchange-protocol-with-secure-device-onboard-crypto-options).

Subsequent messages are covered with an HMAC-SHA that uses the SVK, above, and
encrypted using AES in CBC or CTR mode, as defined in NIST Special Publication
800-38A with the SEK key.

In Secure Device Onboard, the sizes of the SVK and SEK are as follows:


<table>
    <caption>Table 3 - SEK and SVK Sizes</caption>
    <thead>
        <tr>
            <th>Item</th>
            <th>Crypto in Secure Device Onboard 1.0 &amp; Secure Device Onboard 1.1</th>
            <th>Size in Secure Device Onboard 1.0 &amp; Secure Device Onboard 1.1</th>
            <th>Future Crypto</th>
            <th>Future Size</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>SVK</td>
            <td>HMAC-SHA-256</td>
            <td>256 bits</td>
            <td>HMAC-SHA-384</td>
            <td>512 bits</td>
        </tr>
        <tr>
            <td>SEK</td>
            <td>AES-128</td>
            <td>128 bits</td>
            <td>AES-256</td>
            <td>256 bits</td>
        </tr>
    </tbody>
</table>

See section [§](../data-transmission-persistence/#encrypted-message-body) for a description of how encrypted messages are encoded into
JSON and transmitted.

#### Diffie-Hellman Key Exchange Protocol

The following steps describe the Diffie-Hellman key exchange protocol
(DHKEXid15), as part of the verification of the Ownership Voucher:

1.  The Device and Owner each choose random numbers (Owner: a, Device: b), and
    encode these numbers into exchanged parameters A = ga mod p, and B = gb mod

    1.  The values “p” and “g” are chosen from
        [[RFC3526]] , with sizes as follows:

<table>
    <thead>
        <tr>
            <th></th>
            <th>Secure Device Onboard1.0 &amp; Secure Device Onboard1.1 DHKEXid14</th>
            <th>Future Crypto DHKEXid15</th>
            <th></th>
            <th></th>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td>Modulus (p) size</td>
            <td>Generator (g) size</td>
            <td>a &amp; b</td>
            <td>Modulus (p) size</td>
            <td>Generator (g) size</td>
            <td>a &amp; b</td>
        </tr>
        <tr>
            <td></td>
            <td></td>
            <td></td>
            <td>size</td>
            <td></td>
            <td></td>
            <td>size</td>
        </tr>
        <tr>
            <td>DH</td>
            <td>2048</td>
            <td>2</td>
            <td>256 bits</td>
            <td>3072</td>
            <td>2</td>
            <td>768 bits</td>
        </tr>
    </tbody>
</table>

2.  The Owner sends A to the Device as parameter TO2.ProveOPHdr.bo.xA.  
    Note that this parameter is signed by the Owner key from the Ownership
    Voucher, which is proved as trusted later in the TO2 Protocol, but before
    the key exchange completes.

3.  The Device sends B to the Owner as parameter TO2.ProveDevice.bo.xB. This
    parameter is signed with the device attestation key.

4.  The Owner computes shared secret ShSe = B<sup>a</sup> mod p.

5.  The Device computes shared secret ShSe = A<sup>b</sup> mod p.

#### Asymmetric Key Exchange Protocol

The following steps describe the Asymmetric key exchange protocol (ASYMKEX or
ASYMKEX3072), as part of the verification of the Ownership Voucher (here \|\| is
used to indicate binary concatenation). Asymmetric key exchange applies only to
devices that support a RSA-based Ownership Voucher (any of the listed RSA public
key types from Table ‎5 in section [§](../protocol-data-types/#public-key-types)). Sizes are as follows:


<table>
    <thead>
        <tr>
            <th></th>
            <th>Owner &amp; Device Randoms</th>
            <th>MGF Hash Function</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>ASYMKEX</td>
            <td>256 bits each</td>
            <td>SHA256</td>
        </tr>
        <tr>
            <td>ASYMKEX3072 (Future Crypto)</td>
            <td>768 bits each</td>
            <td>SHA256<sup id="fnref:2"><a class="footnote-ref" href="#fn:2">1</a></sup></td>
        </tr>
    </tbody>
</table>

[^2]: 2017-12-06 update: SHA256 may be used with RSA-OAEP under the new crypto
guidelines

1.  Owner allocates a random value called the Owner Random. Owner sends the
    Owner Random to the device as TO2.ProveOPHdr.bo.xA. This value is signed
    with the Owner key, but is not encrypted.

2.  Device allocates a random value called the Device Random. Device encrypts
    the Device Random with the Owner public key using RSA encrypt using Optimal
    Asymmetric Encryption Padding (OAEP) with Mask Generation Function (MGF)
    SHA256 (same for Secure Device Onboard 1.0, Secure Device Onboard 1.1 protocol spec and future
    crypto), as received in TO2.ProveOPHdr.pk. This key is also stored in the
    last entry of the Ownership Voucher; the implementation may obtain it from
    either place.

3.  The encrypted Device Random is sent to the Owner as TO2.ProveDevice.bo.xB.
    This parameter is signed with the Device attestation key.

4.  Owner decrypts TO2.ProveDevice.bo.xB using its Owner Private Key (the same
    private key it used to sign in the TO2.ProveOPHdr message). Note that the
    Owner Private Key must be RSA-based.

5.  The Owner & Device each compute shared secret ShSe = DeviceRandom \|\|
    OwnerRandom

#### ECDH Key Exchange Protocol

The following steps describe the ECDH key exchange protocol (ECDH), as part of
the verification of the Ownership Voucher. ECDH applies only to devices that
support an ECDSA-based Ownership Voucher.

Curve and Random parameters are as follows:

<table>
    <thead>
        <tr>
            <th></th>
            <th>Secure Device Onboard 1.0 &amp; Secure Device Onboard 1.1</th>
            <th>Future Crypto</th>
            <th></th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td></td>
            <td>ECC Curve</td>
            <td>Owner &amp; Device Randoms</td>
            <td>ECC Curve</td>
            <td>Owner &amp; Device Randoms</td>
        </tr>
        <tr>
            <td>ECDH KEX</td>
            <td>NIST P-256 (Gx, Gy), p each 256 bits</td>
            <td>128 bits</td>
            <td>NIST P-384 (Gx, Gy), p each 384 bits</td>
            <td>384 bits</td>
        </tr>
        <tr>
            <td>ECDH KEX on Legacy hardware</td>
            <td>N/A</td>
            <td>NIST P-256</td>
            <td>128 bits</td>
            <td></td>
        </tr>
    </tbody>
</table>

Curve parameters are taken from NIST P-series as above, including p and the base
point (Gx, Gy). **ECC curves allocated for key exchange must be used once
only.**

In the rest of this section, symbol \|\| is used to indicate binary
concatenation, and blen(x) length of x in bytes. The output of blen(x) is a
16-bit unsigned integer (UInt16).

1.  The Device and Owner each choose random numbers (Owner: a, Device: b), and
    encode these numbers into exchanged parameters A = (G<sub>x</sub>, G<sub>y</sub>)\*a mod p, and
    B = (G<sub>x</sub>, G<sub>y</sub>)\*b mod p. A and B are points, and have components (A<sub>x</sub>, A<sub>y</sub>) and
    (B<sub>x</sub>, B<sub>y</sub>), respectively, with bit lengths same as (Gx, G<sub>y</sub>).

2.  The Device and Owner each choose a random number (as per table above), to be
    supplied with their public keys, respectively DeviceRandom, and OwnerRandom.

3.  The Owner sends **ByteArray[blen(A<sub>x</sub>), A<sub>x</sub>, blen(A<sub>y</sub>), A<sub>y</sub>, blen(OwnerRandom),
    OwnerRandom]** to the Device as parameter TO2.ProveOPHdr.bo.xA. Note that
    this parameter is signed by the Owner key from the Ownership Voucher, which
    is proved as trusted later in the TO2 Protocol, but before the key exchange
    completes.

4.  The Device sends **ByteArray[blen(B<sub>x</sub>), B<sub>x</sub>, blen(B<sub>y</sub>), B<sub>y</sub>,
    blen(DeviceRandom),DeviceRandom]** to the Owner as parameter
    TO2.ProveDevice.bo.xB. This parameter is signed with the device attestation
    key.

5.  The Owner computes shared secret Sh = (B\*a mod p), with components (Sh<sub>x</sub>,
    Sh<sub>y</sub>). The Device computes shared secret Sh = (A\*b mod p), with components
    (Sh<sub>x</sub>, Sh<sub>y</sub>). The shared secret ShSe is formed as: Sh<sub>x</sub>
    \|\|DeviceRandom\|\|OwnerRandom (Note that Sh<sub>y</sub> is not used to construct ShSe).

!!!Note
    -   The DeviceRandom and OwnerRandom values are used to increase the entropy in
        the generated keys, in order to reduce the possibility of certain related
        key weaknesses.

    -   The lengths of a, b, DeviceRandom and OwnerRandom are chosen to permit the
        shared secret to source SVK & SEK of appropriate lengths.

    -   In steps 3 and 4, the values of A, B, DeviceRandom and OwnerRandom are
        transmitted within a single ByteArray as length-preceding binary strings. In
        item 3, the first byte is 32[48 future crypto] (=blen(A<sub>x</sub>)), followed by the
        binary bytes of A<sub>x</sub>, followed by 32[48 future crypto] (=blen(A<sub>y</sub>)), followed
        by the binary bytes of A<sub>y</sub>, followed by a byte containing 16[32 future
        crypto] (blen(OwnerRandom)), followed by the binary bytes of OwnerRandom.
        The entire ByteArray is subsequently encoded in base64 for transmission, as
        is usual for ByteArrays.

    -   Compatibility Note: This mechanism is intended to be a standard
        implementation of NIST ECC P-256 or P-384, compatible with other software
        and hardware implementations. Please let us know of any compatibility
        issues. **Note that some popular hardware supports NIST ECC P-256 only
        (For example, ATECC508a). Legacy hardware may also require larger Device and Owner
        randoms when used with larger SEK and SVK for future crypto. Please contact
        the Secure Device Onboard Enablement team for details.**

    -   Shy is not used to compute the shared secret ShSe because it can be derived
        from Shx and the curve equation. Hence it provides no additional entropy.

#### Key Derivation Function

Owner and Device both have shared secret ShSe, computed by one of the above key
exchange protocols. The shared secret ShSe is fed into the Key Derivation
Function defined in NIST Special Publication 800-108, KDF in Counter Mode,
section 5.1.

-   Double vertical bar (\|\|) means binary concatenation, so a\|\|b\|\|c means
    concatenate the bits of a,b,c together.

##### Secure Device Onboard 1.0 and Secure Device Onboard 1.1 Protocol Specification

The following steps 1-4, which continue from the key exchange steps 1-5 described above in [ECDH Key Exchange Protocol](#ecdh-key-exchange-protocol), are
based on ShSe for Secure Device Onboard 1.0 / Secure Device Onboard 1.1, and yield SVK of 256 bits and SEK of 128
bits (see Table ‎3):

1.  KeyMaterial1 =  
    **HMAC-SHA-256[0,(byte)1\|\|"MarshalPointKDF"\|\|(byte)0\|\|"AutomaticProvisioning-cipher"\|\|ShSe]**

2.  KeyMaterial2 =  
    **HMAC-SHA-256[0,(byte)2\|\|"MarshalPointKDF"\|\|(byte)0\|\|"AutomaticProvisioning-hmac"\|\|ShSe]**

3.  SessionEncryptionKey = SEK = KeyMaterial1[0..15] (128 bits, to feed AES128)

4.  SessionVerificationKey = SVK = KeyMaterial2[0..31] (256 bits, to feed
    SHA256)<sup id="fnref:3"><a class="footnote-ref" href="#fn:3">2</a></sup>

[^3]: The SHA256 function takes 256 bits, so we use the KDF to derive 256
    bits. However, the strength of the SVK is assumed to be no more than 128
    bits, because of the entropy used.

!!! Note 
    The operation is HMAC-SHA-256[key, value], so the zero argument above
    indicates a HMAC with key of zero (0). Since HMAC keys are zero padded (See
    ), this should be sufficient to generate a consistent HMAC operation.

The above strings are ASCII with no terminator character (that is, C ‘&bsol;000’
terminator is not included).

##### Future Crypto

The following steps 1-4, which continue from the Key Exchange steps 1-5 described above in [ECDH Key Exchange Protocol](#ecdh-key-exchange-protocol), are
based on ShSe for future crypto, and yield SVK of 512 bits and SEK of 256 bits
(see Table ‎3):

1. KeyMaterial1 =  
    **HMAC-SHA-384[0,(byte)1\|\|"MarshalPointKDF"\|\|(byte)0\|\|"AutomaticProvisioning-cipher"\|\|ShSe]**

2. KeyMaterial2a =  
    **HMAC-SHA-384[0,(byte)2\|\|"MarshalPointKDF"\|\|(byte)0\|\|"AutomaticProvisioning-hmac"\|\|ShSe]**  
   KeyMaterial2b =  
   **HMAC-SHA-384[0,(byte)3\|\|"MarshalPointKDF"\|\|(byte)0\|\|"AutomaticProvisioning-hmac"\|\|ShSe]**  
    (Note the 0/1 byte in the middle of each expression)

3. SessionEncryptionKey = SEK = KeyMaterial1[0..31]  
    (256 bits, to feed AES256)

4. SessionVerificationKey = SVK = KeyMaterial2a[0..47] \|\|
    KeyMaterial2b[0..15]  
    (512 bits to match 512 bit internal state of HMAC-SHA-384)

!!! Note 
    The operation is HMAC-SHA-384[key, value], so the zero argument above
    indicates a HMAC with key of zero (0). Since HMAC keys are zero padded (See
    8), this should be sufficient to generate a consistent HMAC operation.

The above strings are ASCII with no terminator character (that is, C ‘&bsol;000’
terminator is not included).

#### Mapping of Key Exchange Protocol with Secure Device Onboard Crypto Options

Table ‎4 shows the valid choices for key exchange protocol based on choice of
device attestation and owner attestation algorithms selected by the device
manufacturer. The key exchange method may be configured in the device at the
time of manufacturing and not dynamically selected during TO2 protocol.

Note that asymmetric key exchange requires that owner key be RSA based.

The choice of cryptography for the key exchange protocol follows the
cryptography in the Ownership Voucher (Owner key, and other keys in the
Ownership Voucher). Where the Device key and Owner key use different
cryptography, the Device and Owner may need to support additional algorithms to
allow verification and key exchange. We encourage a choice that limits the
software or hardware required in the Device. Note that the difference in
signature verification performance between ECDSA and RSA algorithms might favor
a hybrid approach for some devices.


<table>
    <caption>Table 4 - Key Exchange and Secure Device Onboard Crypto Mapping</caption>
    <thead>
        <tr>
            <th>Device Attestation</th>
            <th>Owner Attestation</th>
            <th>Key Exchange</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>EPID</td>
            <td>RSA2048RESTR</td>
            <td>DHKEXid14/ASYMKEX</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>RSA2048RESTR</td>
            <td>DHKEXid14/ASYMKEX</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>RSA2048RESTR</td>
            <td>DHKEXid14/ASYMKEX (Not a recommended configuration,
    see note)</td>
        </tr>
        <tr>
            <td>EPID</td>
            <td>RSA 3072-bit key</td>
            <td>DHKEXid15/ASYMKEX3072</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>RSA 3072-bit key</td>
            <td>DHKEXid15/ASYMKEX3072 (Not a recommended
    configuration, see note)</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>RSA 3072-bit key</td>
            <td>DHKEXid15/ASYMKEX3072</td>
        </tr>
        <tr>
            <td>EPID</td>
            <td>ECDSA NIST P-256</td>
            <td>ECDH</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>ECDSA NIST P-256</td>
            <td>ECDH</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>ECDSA NIST P-256</td>
            <td>ECDH (Not a recommended configuration)*</td>
        </tr>
        <tr>
            <td>EPID</td>
            <td>ECDSA NIST P-384</td>
            <td>ECDH384</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>ECDSA NIST P-384</td>
            <td>ECDH384 (Not a recommended configuration, see note)</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>ECDSA NIST P-384</td>
            <td>ECDH384</td>
        </tr>
    </tbody>
</table>

---
**Note on not recommended configurations, above**

These configurations have different cryptographic strength between
the Ownership Voucher (Owner key) and the Device key. It is recommended to have
the strongest cryptographic methods that device is capable of for efficiently
verifying both the device key and the Owner key.

---

## The Ownership Voucher

The Ownership Voucher is a structured digital document that links the
Manufacturer with the Owner. It is formed as a chain of signed public keys, each
signature of a public key authorizing the possessor of the corresponding private
key to take ownership of the Device or pass ownership through another link in
the chain.

The voucher artifact described in IETF RFC8366 is different both in
form and function from the Secure Device Onboard Ownership Voucher described here.

The following diagram illustrates an Ownership Voucher with 3 entries. In the
first entry, Manufacturer A, signs the public key of Distributor B. In the
second entry, Distributor B signs the public key of Retailer C. In the third
entry, Retailer C signs the public key of Owner D.

The entries also contain a description of the GUID or GUIDs to which they apply,
and a description of the make and model of the device.

<figure id="fig-OVChain">
<!--    <img src="../img/75ca59cf829237b1ed6db5c553e7187a.emf" alt=""/> -->
    <img src="../img/ownership-voucher-chain.png" alt="">
    <figcaption>Figure 3 - Ownership Voucher Chain</figcaption>
</figure>

The signatures in the Ownership Voucher create a chain of trust from the
manufacturer to the owner. The Device is pre-provisioned (in the Device
Initialize Protocol (DI)) with a crypto-hash of A.PublicKey, which it can verify
against A.PublicKey in the Ownership Voucher. The owner can prove his connection
with the Ownership Voucher (and thus his right to take ownership of the Device)
by proving its ownership of D.PrivateKey. It can do this by signing a nonce or
other ephemeral object, which signature may be verified using D.PublicKey from
the Ownership Voucher.

The last entry in the Ownership Voucher belongs to the current owner. The public
key signed in that entry is the owner’s public key, signed by the previous
owner. We can reasonably call this public key the “Owner Key.”

In the TO2 Protocol, the Owner proves his ownership to the device using a
signature (as above) and an Ownership Voucher that is rooted in A.PublicKey. The
device verifies the hash of A.PublicKey stored in its TEE matches A.PublicKey in
the Ownership Voucher, then verifies the signatures of the Ownership Voucher in
sequence, until it comes to D.PublicKey. The Owner provides the Device separate
proof of D.PublicKey (the “owner key”), completing the chain of trust. Note,
that the only private key needed to prove ownership is that of the Owner. The
public keys in the Ownership Voucher (and the public key hash in the Device) are
sufficient to verify the chain of signatures.

The public keys in the Ownership Voucher are just public keys. They do not
include other ownership info, such as the name of the entity that owns the
public key, what other keys they might own, where they are, and others.

In fact, the Ownership Voucher is maintained only for the purposes of connecting
a particular device with its particular first owner. The entities involved can
and should switch the key pairs they use to sign the Ownership Voucher from time
to time, to ensure that potential attackers cannot use Ownership Vouchers as a
means to map out the flow of devices from factory to implementation. The Secure Device Onboard protocols also help to mask this information, but refreshing public keys is
a useful additional step.

Conversely, if it is desired to have specific knowledge of each of the parties
contributing to the Ownership Voucher, this information might be provided by
hosting X.509 certificates with the same public keys as the Ownership Voucher at
a specific (public or private) site known to the transacting parties. In this
case, the Ownership Voucher can be used as a record of the supply chain. Other,
external, guarantees might be needed to ensure that the Ownership Voucher
contains all the parties in the supply chain.


!!! Note 
    The Ownership Voucher-signing operation is not related to the device
    attestation operation – that is, a device can use RSA or ECDSA for Ownership
    Voucher chain signing, independent of whether it uses Intel EPID or ECDSA
    for device attestation.


### Building the Ownership Credential & Ownership Voucher

Secure Device Onboard Ownership Proxies contain information needed by the
Owner:

-   Rendezvous information sufficient to contact the Rendezvous Server

-   The GUID of the device

-   A Device Info string from the manufacturer that identifies the device model

The Ownership Voucher is linked to the Ownership Credential, so the first part
of the “tool chain” to build an Ownership Voucher builds the Ownership
Credential.

As shown in sections ‎[§](../protocol-data-types/#composite-types) & [§](../data-transmission-persistence/#error-message), the Ownership Credential and Ownership Voucher are formatted as persisted messages using the distinguished JSON encoding. It is allowed to reformat them for storage and transmission. Since the Ownership Credentials are not signed or otherwise protected, it is required that they be suitably protected when stored or transmitted. For example, they might be signed using an available asymmetric key and sealed with an available secret key.

Based on the Ownership Credential and a public key (B.PublicKey in the example
above), an Ownership Voucher of 1 segment may be created. The signing key for
the Ownership Voucher is a private key whose public key matches the hash in the
Ownership Credential (A.PrivateKey, above). The GUID and DeviceInfo in the
Ownership Voucher header must also match the hash in the Ownership Voucher
entry.

A secret is created in the Device TEE in the DI Protocol. This secret is used to
create a HMAC of the Ownership Voucher header. The HMAC can only be verified in
the same Device TEE, and is used to detect a device that has been reprogrammed
after it left the factory. The HMAC size is given in Table ‎5.


<table>
    <caption>Table 5 - Cryptographic Sizes for Ownership Voucher</caption>
    <thead>
        <tr>
            <th>Item in Ownership Voucher</th>
            <th>Secure Device Onboard 1.0 &amp; Secure Device Onboard 1.1</th>
            <th>Future Cryptography</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>HMAC in Ownership Voucher</td>
            <td>HMAC-SHA-256, based on 256-bit randomly allocated secret stored in Device</td>
            <td>HMAC-SHA-384, based on 512-bit randomly allocated secret stored in Device</td>
        </tr>
        <tr>
            <td>Public keys in Ownership Voucher (all must have same size and type)</td>
            <td>RSA-2048 with restricted exponent (type RSA2048RESTR) Or ECDSA NIST P-256</td>
            <td>RSA with 3072-bit key</td>
        </tr>
        <tr>
            <td></td>
            <td></td>
            <td>(type RSA_UR) Or ECDSA NIST P-384</td>
        </tr>
    </tbody>
</table>

The key pair used for the Ownership Voucher may be chosen based on the available
cryptography in the Device in question at manufacturing initialization time. The
cryptographic strength is given in Table ‎5.

Legacy devices may be permitted to use smaller cryptographic sizes. Contact the
Secure Device Onboard Enablement team for more details.

Subsequently the Ownership Voucher may extend as follows:

-   Required:

    -   Ownership Voucher with N segments, N &ge; 1

    -   Owner Key Pair - private and public key  
        This is the public key in segment N, and its corresponding private key. Keys
        for earlier segments are not needed.

    -   The GUID of the Device

    -   The DeviceInfo String, “d” in the Ownership Voucher header
        (OwnershipProxy.oh.d)

    -   The Public Key for the new segment - the next owner’s key  
        The private key corresponding to this public key is used either to provision
        the device using the protocols described in this document, or to extend the
        Ownership Voucher further.

-   Procedure

    -   All hashes are computed as per Table ‎5, except for legacy hardware, as
        informed by contacting the Secure Device Onboard Enablement team.

    -   Hash is computed of segment N

        -   For segment 1, the hash covers the Ownership Voucher header and the HMAC
        that protects it (“oh” tag value \|\| “hmac” tag value)

    -   A new segment is created, containing:

        -   The public key for the new segment

        -   Hash[GUID \|\| DeviceInfo] (the two values concatenated)

        -   The hash of segment N

    -   The new segment is then signed using the Owner key from Segment N, and
        appended to the ownership voucher, to become segment N+1; its signed public
        key becomes the new (next) Owner key.

    -   Note that the public key signed in segment N verifies the signature in
        segment N+1. The public key in the Ownership Voucher header verifies the
        signature in the first segment, segment 0.

    -   Each key in the Ownership Voucher must have the same public key type (see
        Table 5 in section [§](../protocol-data-types/#public-key-types)) and encoding (see Table 6 in section [§](../protocol-data-types/#public-key-encodings)) as appears in the Ownership
        Voucher’s header (For example, all RSA2048RESTR, all RSA_UR, all ECDSA P-256 or all
        ECDSA P-384). This ensures that a Device with limited crypto capabilities
        can verify all the signatures.

### Verifying the Ownership Credential

The Ownership Credential in the Device TEE must be stored securely in a manner
that prevents and/or detects modification. Write-once memory, where available,
is a useful assistive technology. The HMAC secret stored in the Device TEE is
used as a fall-back where such technologies are not available.

To the extent possible, the HMAC secret should be linked to the other
credentials, so that modifying any credential invalidates the HMAC secret.

The HMAC secret is the only Device credential that requires confidentiality.

### Validation of Device Certificate Chain

In case of ECDSA device attestation, device certificate chain is included in the
Ownership Voucher. The device certificate chain is a hierarchical list of
certificates, starting from device certificate to intermediate CAs to root CA,
where each certificate is signed by the next certificate in the list until the
root CA which is self signed. The device certificate contains the public key
corresponding to the ECDSA private key that is in the device TEE, and is used by
the device to attest its identity in TO1 and TO2 protocols. When the Owner
receives an Ownership Voucher, it may validate the device certificate chain to
determine if it can trust the device certificate. If the validation fails, the
Owner may decide to reject the device. Since a problem in device certificate
chain may result in a large batch of devices to be rejected by the Owner, the
manufacturing tool must perform some basic validation during the DI protocol per
the following requirements:

-   All certificates in the chain must be in X.509 format.

-   Certificate path validation as per RFC 5280, Section 6.1 (Basic Path
    Validation) must be successful. For this, the tool may use standard APIs
    such as Java Class CertPathValidation (PKIX algorithm).

-   For device certificate (leaf certificate in the chain), the following must
    be validated:

-   Public Key Algorithm must be ECDSA (id-ecPublicKey).

-   Length of Public Key must be either 256-bit or 384-bit.

-   The ECDSA curve parameters for the Public Key must be set to either
    secp256r1 (NIST P-256) or secp384r1 (NIST P-384), based on the public key
    length. See RFC 5480 for more details.

-   If Key Usage extension is present in the device certificate, then it must
    allow Digital Signature.

### Verifying the Ownership Voucher

The Ownership Voucher is stored as a persisted message to be used in the TO0 and
TO2 Protocols. It must be verified several ways:

-   When being read from storage or inside a protocol transmission, the
    Ownership Voucher must be internally verified to make sure it has not been
    tampered with.

-   If the Ownership Voucher is extended or transmitted, the owner must prove
    that he controls the Owner private key.

-   The Device receiving the Ownership Voucher must verify it against the
    ownership credential and verify the HMAC in the Ownership Voucher using the
    secret stored in the device.

#### Ownership Voucher Internal Verification

Internal verification should be performed whenever the Ownership Voucher is read
from its persisted storage or received in a protocol transmission (for more
information on these messages, please see section [§](../detailed-protocol-description/#to0ownersign-type-22) and sections [§](../detailed-protocol-description/#to2hellodevice-type-40) through section [§](../detailed-protocol-description/#to2opnextentry-type-43)).

To verify the internal consistency of the ownership voucher, the following steps
are performed:

-   The device spec is verified to match in all segments.

-   The GUID is verified to match in all segments.

-   The signature of each segment is verified against the signed public key of
    the previous segment.

-   The first segment is verified against the public key Ownership Voucher
    header (oh.pk).

-   When verifying an encoded version of the Ownership Voucher, the hash stored
    in each entry can be verified to match the hash of the previous entry’s
    encoding. The first entry matches the hash of the encoding of the Ownership
    Voucher header as described in section [§](../protocol-data-types/#composite-types): Composite Types.

#### Owner Verification against the Owner Key

When the Owner reads the Ownership Voucher from storage, it must verify that its
Owner key pair corresponds to the signed key in the last segment. This is
accomplished using one of these methods:

-   If the Owner has assurance that its stored public and private key are a
    pair,<sup id="fnref:4"><a class="footnote-ref" href="#fn:4">3</a></sup> the stored public key may be compared against the signed public
    key in the last segment of the Ownership Voucher.

[^4]: Alternately, owner can prove that the stored public and private key
    are a pair by signing a nonce with the private key and verifying it using
    the public key.

-   The Owner may sign a nonce using the stored private key, and verify the
    signature twice, using both the stored public key and the signed public key
    in the last segment of the Ownership Voucher.

#### Owner Verification of Device Certificate Chain

When an Owner receives the Ownership Voucher, the Owner must decide whether to
trust or to distrust the device certificate chain. This decision is typically
based on an external trust relationship with the device’s supply chain. It can
also be aided by cryptographic verification, but such verification cannot
replace external trust. The following cryptographic steps are recommended:

-   The certificates and signature chain of OwnershipProxy.dc are verified.

-   OCSP information is obtained from each certificate, and where present, the
    OCSP protocol is run to determine whether the Device key is revoked. If so,
    the Ownership Voucher (and the Device) are rejected, and Secure Device Onboard is not
    possible without re-programming the Device (For example, using manual or automatic
    TCB recovery).

-   If possible, one or more of the certificate chain CA’s should be previously
    trusted by the Owner. If not, the Owner uses its own judgement as to whether
    to accept the Ownership Voucher based on other business criteria, such as
    the trust of supply chain partners.

#### Receiver Verification of Owner

When the Owner transmits the Ownership Voucher to the Rendezvous Server or to
the Device, the receiver must verify the internal structure of the Ownership
Voucher, and also verify the signature that the Owner provides in TO0.OwnerSign
(section [§](../detailed-protocol-description/#to0ownersign-type-22)) and the last transmission of TO2.ProveOpHdr (section [§](../detailed-protocol-description/#to2proveophdr-type-41)) against the
Owner key (the public key signed in the last entry of the Ownership Voucher). In
the case of the Device, the Ownership Credential key must also verify the
signature in the first entry of the Ownership Voucher.

In this last situation, the public key to verify the signature appears as the
first entry of the Ownership Voucher, and a hash of this public key is in the
Ownership Credential. The signature must be checked by the public key, and the
public key’s hash must be checked against the Ownership Credential.

The TO2 Protocol transmits the Ownership Voucher in pieces. As an aid to
constrained Devices, the receiver can process the Ownership Voucher pieces in
order, without needing to ever store the entire Ownership Voucher. Pseudo code
for this is provided in section [§](../detailed-protocol-description/#to2opnextentry-type-43).

#### Service Verification of the Ownership Voucher

The Secure Device Onboard protocols do not supply the Rendezvous Service with a mechanism
for determining the trust of the Ownership Voucher. It is desirable for the
Rendezvous Service to be able to trust one or more of the keys in the Ownership
Voucher. This can be accomplished using a back channel to supply public keys (or
public key hashes) to the Rendezvous Service as they are created by cooperating
supply-chain entities. It is imperative, for security reasons, that these keys
be stripped of any associated data that identify the key holder before they are
configured into the Rendezvous Service.

