# Detailed Protocol Description

This section defines protocol messages and interactions. Protocol message names
match the table, in section [§](../data-transmission-persistence/#message-types): Message Types*.

The notation for each message is based on JavaScript\* Object Notation (JSON).
The message is presented as a message header and a JSON\* object as body. The body
uses comments to indicate the meaning of particular JSON\* elements.

Note that these comments and the whitespace are *illegal to be transmitted*
within the JSON* encoding described in section [§](../protocol-description/#json-distinguished-encoding), and are provided for
explanation only.


Composite types described in the tables in section [§](../data-transmission-persistence/#message-types) are used freely. These
are also expanded out in some cases, to allow comments on the internal members.
In this case, the opening bracket should contain a comment to indicate the
composite type.


## General Messages

### Error - Type 255

Message Body:

```json
{ #Error message body
    "ec": Uint16, # Error code
    "emsg": UInt8, # Message ID of the previous message
    "em": String # Error string
}
```

Message Meaning:

The error message indicates that the previous protocol message could not be
processed. The error message is described in detail in section ‎[§](../data-transmission-persistence/#error-message).

## Persisted Messages 

Persisted messages are messages that are stored on non-volatile media and
retrieved later. They may also be transmitted outside of the Secure Device Onboard
protocols, such as in e-mail messages, and act as an interchange format.

Since all Secure Device Onboard messages are printable ASCII, persisted messages may be
generated and parsed in exactly the same way as transmitted messages, with only
their message type to distinguish them as persisted.

Persisted messages always contain the JSON* 4-element sequence encapsulation, as
described in section [§](../data-transmission-persistence/#transmission-of-messages-over-a-stream-protocol-and-persisted-messages).

The 4-element JSON* encapsulation always appears for these messages when they are
stored. For example, if the PM.PublicKey type’s body is the 16 characters:

```json
{"pk":[0,0,[0]]}
```

(This is a null public key, not very useful.)

Then the persisted message will actually be:

```json
["001d",4,7,{"pk":[0,0,[0]]}]
```

Where the “001d” in the 4-character length field indicates that the entire
message is 29 bytes (=0x1d) long.

### PM.CredOwner, Type 1

Stored in Device TEE

This is the Ownership Credential that is stored in the device TEE during the
Device Initialize Protocol (DI), and updated in Transfer Ownership 2 Protocol
(TO2).

Message Body:

```json
{ # OwnerBlock
    "pv": UInt16,
    "pe": UInt8,
    "g": Guid,
    "r": RendezvousInfo,
    "pkh": Hash
}
```

Message Meaning:

The “pv” parameter specifies the protocol version as a UInt16, in the same
format as the message header. The protocol version must be persisted whether the
message header information is available or not (For example, in the Device TEE).

The “pe” parameter specifies the key encoding used in this credential, and in
ownership proxies based on this credential.

The GUID parameter “g” is the current device GUID, to be used for the next
ownership transfer.

The RendezvousInfo parameter “r” contains instructions on how to find the Secure
Device Onboard Rendezvous Server.

The Public Key Hash “pkh” is a hash of the Owner’s public key, which must match
the first entry of the Ownership Voucher used to transfer ownership for this
device. The recommended default hash to use is SHA-256 (Secure Device Onboard 1.0 & Secure Device Onboard 1.1 protocol spec; SHA-384 for future crypto).

### PM.CredMfg, Type 2

Stored in Device TEE:

This is the Manufacturing Credential that is stored in the device TEE during the
Device Initialize Protocol (DI). It is never changed afterwards.

Message Body:

```json
{ # ManufacturerBlock
    "d": String
}
```

Message Meaning:

The DeviceInfo parameter “d” is the manufacturer’s DeviceInfo field. This may be
used to identify the device model.

### PM.OwnershipProxy, Type 3

The [Ownership Voucher] (previously called as Ownership
Proxy) is used to convey the trust of the device in the factory to the new
owner.

The OwnershipProxy structure is shown in Table ‎1:


<table>
    <caption>Table 1 - OwnershipProxy Structure</caption>
    <thead>
        <tr>
            <td>
<b>OwnershipProxy</b>
```json
{
    "sz": UInt8,	# number of entries
    "oh": {		
	"pv": UInt16,	# protocol version
	"pe": UInt8,	# public key encoding
	"r": Rendezvous,  # rendezvous
	"g": GUID,	# guid
	"d": String,	# DeviceInfo
	"pk": PublicKey,	# mfg public key
	"hdc": Hash	# hmac[secret, “oh”]    # Hash of device certificate chain, absent if using Intel<sup>®</sup> EPID
    },
    "hmac": HMac,	
    “dc”: CertChain,	# Device certificate chain, absent if EPID
    "en": [# entries.
	OwnershipProxyEntry[0],
	…
	OwnershipProxyEntry\[SZ-1]
    ]#(Actually, zero entries are permitted, and looks like "en":[],)
}
```
<b>Note:</b> When "sz" is zero, "en" is [].

	    </td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
            
<b>CertChain</b>

```json
[
    type(Uint8),	# format of certificate entries (1==x509)
    numEntries(Uint8),	# number of certificate entries
    [			# Array of certs from Device to CA, each signed by next
        Cert[0],
        …
        Cert[numEntries-1]
    ]
]
```
	    </td>
        </tr>
        <tr>
            <td>
            
<b>Cert</b>

```json
[
    length(Uint16),	# number of bytes in certificate
    certBytes(ByteArray)  # certificate data in DER encoded to ByteArray format
]
```
	    </td>
        </tr>
        <tr>
            <td>
            
<b>OwnershipProxyEntry</b>

```json
{
    bo:{
        "hp": Hash,	# hash of previous entry (only the “bo”)
        "hc": Hash,	# hash[GUID||DeviceInfo] in header*
        "pk": PublicKey	# public key being signed
    },
    "pk": PKNull,	# place holder (null pub key)
    "sg": Signature	# signature
}
```
<b>Note: <b> GUID||DeviceInfo indicates the bitwise concatenation of the "g" and
"d" fields from the ownership proxy header.

	    </td>
        </tr>
    </tbody>
</table>

Message Meaning:

The “oh” field contains header information, a copy of which is stored in the
Device (the Device stores only a hash of the public key “oh.pk”). The “oh”
field’s contents are hashed into “hmac” by the device TEE and combined with a
secret, which is only stored in the device TEE.

-   “oh.pv” is the protocol version (major version \* 100 + minor version).

-   “oh.pe” is the protocol encoding used in all Ownership Voucher public keys.

-   “r” is the rendezvous info for connecting to the Rendezvous Server.

-   “g” is the current GUID of the device.

-   For ECDSA device attestation, the device certificate chain is present in the
    Ownership Voucher as OwnershipProxy.dc (device certificate). This is of type
    CertChain. When the device uses an Intel<sup>®</sup> EPID root of trust,
    OwnershipProxy.dc is not present.

-   “pk” is the public key of the device’ initial owner (For example, the
    manufacturer).

The “en” tag contains the Ownership Voucher entries, in order. If there are no
entries (“sz” is zero) the “en” tag is an empty array ([]).

### PM.PublicKey, Type 4

This message format is used by each successive owner in the supply chain to identify his public key to the previous owner. The previous owner signs the public key (and other information) to extend the ownership voucher. See section [§](../protocol-description/#the-ownership-voucher): The Ownership Voucher.


Message Body:

```json
{
    "pk": PublicKey
}
```

Message Meaning:

For a description of the various public key formats, see section [§](../protocol-data-types/#public-key-types): Public Key Types.

For a description of how the public key of the new Owner is used to extend the
Ownership Voucher, see section [§](../protocol-description/#building-the-ownership-credential-ownership-voucher): Build The Ownership Voucher.

### PM.ServiceInfo, Type 5

Stored in Device Filesystem:

This message format is used to save the key-value pairs that are sent as part of
the TO2.ReceiveDeviceInfo and TO2.SendSetupInfo messages.

Message Body:

```json
{
    "k1": "v1", # note: k1 must be ASCII
    "k2": "v2", # note: k2 must be ASCII
    …
    "kN": "vN" #note: kN must be ASCII
}
```

Note -  k1, k2, … kN must all be unescaped ASCII. For example: "abc" is legal, but
    "a&bsol;u0062c" is illegal. This does mean that keys cannot have special
    characters in them. However, values can have escaped characters, as needed,
    to support the full Unicode set.

Message Meaning:

ServiceInfo is a set of key-value pairs that is used in a negotiation between
the Device and the Owner during the TO2 Protocol. A set of key-value pairs, the
Device ServiceInfo is first transmitted from the Device to the Owner. Then a set
of key-value pairs, the Owner ServiceInfo, is sent from the Owner to the Device.

Definitions of ServiceInfo key-value pairs is given in section [§](../protocol-data-types/#serviceinfo-and-management-service-agent-interactions).

### PM.DeviceCredentials, Type 6

Stored in Device Filesystem:

This message format is suggested to store the device state for implementations
that use a filesystem instead of a TEE. It may also be signed or encrypted
(sealed) in these implementations, to improve security. A given implementation
may also use another, more convenient, format.

Message Body:

```json
{
    "ST": UInt8, # State
    "Secret": ByteArray,
    "M": {#ManufacturerBlock
	"d": String
    },
    "O": {#OwnerBlock
	"pv": UInt16,
	"pe": UInt8,
	"g": Guid,
	"r": RendezvousInfo,
	"pkh": Hash
    }
}
```

Message Meaning:

“ST” is the Secure Device Onboard state, as defined in the Secure Device Onboard architectural
specification.

“M” is the manufacturer state.

“O” is the owner state.

## Device Initialize Protocol (DI) 

The Device Initialize Protocol (DI) serves to set the manufacturer and owner of
the device in the TEE. It is assumed to be performed at device manufacture time.

This protocol uses a Trust On First Use (TOFU) trust model, consistent with the
Secure Device Onboard assumption that the manufacturing environment is trusted.

It would be possible to implement a more restricted trust model for the DI
Protocol by embedding a public key into the TEE, with the TEE owner providing
signing (or at least CA) services to the manufacturer.

The DI Protocol runs between a manufacturing support station, which contains the
Secure Device Onboard Manufacturing Component, and the Device TEE.

The Device is assumed to be running with some other kind of support software,
which is able to access the TEE and provide communications services for it. For
example, the device may be PXE-booted into a RAM-based Linux* system, with
features to access the TEE. It is assumed that this software is able to
determine from its environment the IP address for the manufacturing support
station.

<figure id="fig-DIProtocol">
<!--    <img src="../img/fa8231781c209a6f95a114522f469766.emf" alt=""/> -->
    <img src="../img/di-protocol-bounce.png" alt="">
    <figcaption>DI Protocol Diagram</figcaption>
</figure>

### DI.AppStart, Type 10

From Device TEE to Manufacturer:

The App Start message starts talking to the TEE application to start.
Downloading, verifying, and starting the TEE application is outside the scope of
this document.

Message Format:

```json
{
    "m": String
}
```
<!-- Geof: there was a typo in the version of the spec we contributed.  It contains just a string called 'm', no pv. -->

Message Meaning:

Start the process of taking initial ownership of the device.

If available, the device may include a serial number or other identifying mark
from the hardware in this message, using the “m” tag. This is intended to help
the manufacturing station to index Secure Device Onboard information with other information
available to the manufacturer. If no such information is available, the “m” tag
is sent with an empty string.

The manufacturing station must always be able to handle a device that sends an
empty string for tag “m.”

### DI.SetCredentials, Type 11

From Manufacturer to Device TEE:

Message Format:

```json
{
    "oh":{# See Ownership Voucher “oh” tag.
	"pv": UInt16,
	"pe": UInt8,
	"r": Rendezvous,
	"g": GUID,
	"d": String,
	"pk": PublicKey,
	"hdc": Hash # Absent if using Intel EPID
    }
}
```

Message Meaning:

The manufacturing station sends credentials to the Device TEE. The credentials
in "oh" are identical to the “oh” field of the Ownership Voucher (see section [§](#pmownershipproxy-type-3)). Some
additional credentials allow the original manufacturer of the device to be
determined across future ownership transfers.

When the device uses ECDSA method for device attestation, the manufacturing
station will compute the Hash of the device certificate chain (provided by the
manufacturer to the manufacturing tool) and include the Hash as “oh.hdc” in the
message. When the device uses Intel<sup>®</sup> EPID root of trust, this field must not be
present.

The manufacturing station typically will use the “m” tag (see section [§](#diappstart-type-10)) to determine
information for “oh.r,” “oh.d” and “oh.pk.” The “oh.g” field (GUID) shall be a
secure-randomly created unique identifier and not derived in any way from
device-specific information to ensure the privacy of the protocol. The Device
TEE allocates a secret, stores this information in the Device TEE as
PM.DeviceCredentials, along with the secret. The public key oh.pk is stored as a
hash of the public key oh.pk. The Device does not have to use the persisted
message format for PM.DeviceCredentials, as long as the information can be
accurately retrieved on demand.

The Device TEE also computes an HMAC based on the above secret and the entire
contents of this message body (including the brace brackets). This HMAC is used
in the next message.

### DI.SetHMAC, Type 12

From Device TEE to Manufacturer:

Message Format:

```json
{
    "hmac": Hash
}
```

Message Meaning:

The device TEE returns the HMAC of the internal secret and the
DI.SetCredentials.oh tag, as mentioned above. The manufacturer combines this
HMAC with its own transmitted information to create an Ownership Voucher with
zero entries.

### DI.Done, Type 13

**From Manufacturer to Device TEE:**

Message Body:

```
-no-body-
```

Message Meaning:

Indicates successful completion of the DI protocol. This message must not be
sent before the Credentials associated with the device (see section [§](#disetcredentials-type-11)) is
recoverably persisted in the manufacturing backend to prevent the release of an
unusable device due to loss of its Ownership Voucher.

Upon receive of this message, the device persists all information associated
with the Device Initialization protocol.

## Transfer Ownership Protocol 0 (TO0) 

The function of Transfer Ownership Protocol 0 (TO0) is to register the new
owner’s current Internet location with the Rendezvous server under the GUID of
the device being registered. The Rendezvous server negotiates a length of time
during which it will remember the new owner’s current IP address. If the new
owner does not receive a device transfer of ownership within this time, it must
re-connect to the Rendezvous server to repeat Transfer Ownership Protocol 0.

The protocol begins when the Owner Client opens a connection to the Rendezvous
Server as is given in the Ownership Voucher.

The preferred protocol to use is TLS with server authentication only. The
necessary client authentication is provided by the ownership voucher.

<figure id="fig-TO0Protocol">
<!--  <img src="../img/823854db0671fd8b326463c50c629f54.emf" alt="TO0 Protocol"/> -->
  <img src="../img/to0-protocol-bounce.png" alt="TO0 Protocol">
  <figcaption>TO0 Protocol Diagram</figcaption>
</figure>

### TO0.Hello, Type 20

From New Owner Client to Rendezvous Server

Message Format:

```
-No body-
```

Message Meaning:

Initiates the TO0 Protocol, requests a Hello Ack nonce.

### TO0.HelloAck, Type 21

From Rendezvous Server to New Owner Client

Message Format:

```json
{
    "n3": Nonce # nonce #3
}
```

Message Meaning:

Requests proof of the ownership voucher.

### TO0.OwnerSign, Type 22

From New Owner Client to Rendezvous Server

Message Format:

```json
{
    "to0d": { # covered in signature
	“op”: OwnershipProxy, # Ownership Voucher (complete)
	"ws": UInt32, # how many seconds to wait.
	"n3": Nonce # Freshness of signature
    },
    "to1d": {
	"bo": { #sign with “Owner key”
	    "i1": IPAddress, # IP address where we are waiting
	    "dns1": String, # DNS address where we are waiting (alternative)
	    "port1": UInt16, # TCP/UDP port number
	    "to0dh": Hash # Hash(to0d object, brace to brace)
	},
	"pk": PKNull,
	"sg": Signature # signed with “Owner key” == last public key in “op”.
    }
}
```

Message Meaning:

The new owner demonstrates its credentials for a given GUID by providing the
Ownership Voucher and signing with the Owner Key. In addition, the owner
provides the Internet location where it is waiting for a Device to connect
(fields: “i1”, “dns1”, “port1”) and upper bound of how long it is willing to
wait (field: “ws”). The wait time is negotiated with the server, see
TO0.AcceptOwner.ws. After the negotiated wait time passes, the owner must re-run
the TO0 Protocol to refresh its mapping.

The Ownership Voucher is given in “to0d.op” as a single object. The Ownership
Voucher must have at least one entry, such as, to0d.op.bo.sz &gt; 0, or the
Rendezvous Service must drop the connection.[^21] The Rendezvous Service may
also restrict the maximum number of entries it is willing to accept, to prevent
DoS attacks. The current recommended maximum is ten entries (to0d.op.bo.sz &gt; 0
&amp;&amp; to0d.op.bo.sz &le; 10).

[^21]: The Rendezvous Service cannot verify an ownership voucher with zero
entries.

The encoding of this message is divided into two JSON\* objects: “to0d” and
“to1d”, which are linked by the hash “to0dh” inside of “to1d”. The object “to0d”
contains fields that are only used in the TO0 Protocol, and the object “to1d”
contains fields that are *also* used in the TO1 Protocol.

The fields in to0d are:

<table>
    <tbody>
        <tr>
            <td>to0d.op:
	    </td>
	    <td>
The entire Ownership Voucher. The GUID is given in to0d.op.oh.g. The
Owner Key is given in to0d.op.en[to0d.op.sz-1].bo.pk (that is, the “bo.pk” in the
last OwnershipProxyEntry). The Owner key is used to verify to1d.sg.
	    </td>
	</tr>
        <tr>
            <td>to0d.ws:
	    </td>
	    <td>
The wait time offered by the Owner, which is adjusted and confirmed in
TO0.AcceptOwner.ws.
	    </td>
	</tr>
        <tr>
            <td>to0d.n3:
	    </td>
	    <td>
A copy of TO0.HelloAck.n3, used to ensure the freshness of the
signature in TO0.to1d.sg.
	    </td>
	</tr>
    </tbody>
</table>

The to1d object is a signed “blob” that indicates an Internet address where the
Device can find Owner for the TO2 Protocol. The entire object is stored by the
Rendezvous Service and returned verbatim to the Device in the TO1 Protocol. See
how this value is verified by the Device in section [§](#to0acceptowner-type-25).

The fields in to1d are:

<table>
    <tbody>
        <tr>
            <td>to1d.bo.i1:
	    </td>
	    <td>
An internet address where the Owner is listening for a TO2
connection. It may be 0.0.0.0 if only “dns1” matters. Since the to1d value
has a time limit associated with it (“ws”), the server may use the Internet
address to create a temporary address that is harder to map to its identity.
If both DNS and IP address are specified, the IP address is used only when
the DNS address fails to resolve.
	    </td>
	</tr>
        <tr>
            <td>to1d.bo.dns1:
	    </td>
	    <td>
A DNS name where the Owner is listening for a TO2 connection.
Any IP address resolved by the DNS name must be equivalently able to process
the TO2 connection. A null string (“”) may be used if only the “i1” value
matters.
	    </td>
	</tr>
        <tr>
            <td>to1d.bo.port1:
	    </td>
	    <td>
A TCP port where the Owner is listening for a TO2 connection.
A value of zero (0) indicates that the default port (80 for HTTP or 443 for
HTTPS) is used.
	    </td>
	</tr>
        <tr>
            <td>to1d.bo.to0dh:
	    </td>
	    <td>
A SHA256 or SHA384 hash of the TO0.OwnerSign.to0d object (from brace
to brace, inclusive). The Rendezvous Server must verify that to0dh matches
the hash of the to0d object (from brace to brace). Otherwise, the Rendezvous
Server shall end the connection in error.
	    </td>
	</tr>
        <tr>
            <td>to1.pk:
	    </td>
	    <td>Always PKNull.
	    </td>
	</tr>
        <tr>
            <td>to1.sg:
	    </td>
	    <td>Signature of to1d.bo with the Owner key from to0d.op.
	    </td>
	</tr>
        <tr>
            <td>
	    </td>
	    <td>
	    </td>
	</tr>
    </tbody>
</table>


It is preferred that the Rendezvous Service has a basis on which to trust at
least one public key within the Ownership Voucher. For example, the manufacturer
who ran the DI protocol to configure the Device, thereby choosing the Rendezvous
Service, may register key hashes with the Rendezvous Service to establish such a
trust. The Owner may register its own keys additionally, or as an alternative.
An intermediate signer of the Ownership Voucher might act as a national point of
entry, using its keys to establish trust for devices in the Rendezvous Service
as they arrive in country.

A given Rendezvous Service may choose to reject Ownership Proxies that are not
trusted.

If the Rendezvous Service has no basis on which to trust the Ownership Voucher,
it must apply its own internal policies to protect itself against a DoS attack,
but may otherwise safely provide the Rendezvous Service (that is, it can allow the
TO0 and TO1 Protocols to succeed). This behavior is acceptable because the TO2
Protocol is able to verify the to1d “blob” defined in this message. However,
such a Rendezvous Service must ensure that untrusted Ownership Proxies cannot
degrade the service for trusted Ownership Proxies. This may be accomplished
through hard limiting of resources, or even allocating a trusted- and
non-trusted version of the service.

The Rendezvous Server needs to verify that the signature on this message is
verified by the public key on the last message of the ownership voucher, such as
by saving the public key transmitted and verifying it is the same public key.

In case of ECDSA device attestation method, Rendezvous Service must verify the
binding of the certificate to the Ownership Voucher (verify the certificate
chain hash). It is the only non-owner entity which can do this. It is
recommended that the Service should also do revocation check for the certificate
chain.

### TO0.AcceptOwner, Type 25

From Rendezvous Server to New Owner Client

Message Format:

```json
{
    "ws": Uint32 # waitSeconds;
}
```

Message Meaning:

Indicates acceptance of the new Owner. The Rendezvous Server will associate GUID
with the new owner’s address information for the waitSeconds seconds.
WaitSeconds may not exceed NewOwnerHello.waitSeconds, but it may be less.

If the GUID indicated in:

TO0.OwnerSign.to0d.bo.op.oh.g

is already associated with another IP address, the Rendezvous Server retargets
this association as specified in this protocol.

The New Owner Client can drop the connection after this message is processed.

If the new Owner does not receive a Transfer Ownership connection from a Device
within waitSeconds seconds, it must repeat Transfer Ownership Protocol 0 and
re-register its GUID to address association.

When the new Owner is actively changing its address from time to time (For example, to
mask its identity), the frequency of changing address dictates the magnitude of
WaitSeconds. Otherwise, the negotiation depends on the frequency at which the
new owner wishes to refresh the server, traded off with the server’s need to
remember many GUID associations.

!!! Note
    The Rendezvous Server has no sure way to know when a device ownership is
    successful or fails, since it is not party to the TO2 Protocol. This is
    intended to make it harder for an intruder who is monitoring the Rendezvous
    Server to trace a device, even by the Secure Device Onboard GUID (which is replaced in
    the TO2 Protocol). Thus the Rendezvous Server may arrange to keep the
    timeouts short enough that it does not have to keep every Secure Device Onboard
    transaction ever created in its database. We imagine a timeout of a day or
    two, or perhaps a week or two.

!!! Note
    The type for this message is 25, which is not contiguous with other message
    types. Earlier versions of this section incorrectly gave the type as 23.

## Transfer Ownership Protocol 1

Transfer Ownership Protocol 1 (TO1) finishes the rendezvous started between the
New Owner and the Rendezvous Server in the Transfer Ownership Protocol 0 (TO0).
In this protocol, the Device TEE communicates with the Rendezvous Server and
obtains the IP addressing info for the (potential) new Owner. Then the Device
may establish trust with the new Owner by connecting to it, using the TO2
Protocol.

!!! Note
    The Transfer Ownership Protocols 0 and 1 serve only to get the Device the IP
    addressing information for a potential Owner candidate—no trust is conveyed
    in these protocols.

When possible, the TO1 Protocol should arrive at the Rendezvous service under
HTTPS to protect the privacy of the Owner. It is possible that intermediate
stages of the protocol are run under HTTP, such as from a sensor to a gateway or
from a management engine to an OS user process.

If it is NOT possible to use HTTPS to protect the TO1 Protocol, the Owner may
also take measures to protect its privacy:

-   The Owner may use a private IP address (For example, IPv6 privacy address) and
    refresh the address periodically, to make it more difficult for an attacker
    to glean information from the rendezvous address.

-   The Owner may use a multi-tenant model, where the actual Owner of the Device
    does not relate to the IP address or DNS name of the Owner.

<figure id="fig-T01Protocol">
<!--    <img src="../img/0557bfdb787cc077abdfaf757a06eb35.emf" alt="TO1 Protocol"/> -->
    <img src="../img/to1-protocol-bounce.png" alt="TO1 Protocol">
    <figcaption>Transfer Ownership Protocol 1 (TO1)</figcaption>
</figure>

### TO1.HelloSDO, Type 30

From Device TEE to Rendezvous Server:

Message Format:

```json
{
    "g2": GUID, # device GUID.
    "eA": SigInfo
}
```
<!-- Geof: typo, g2 missing -->

Message Meaning:

Establishes the presence of the device at the Rendezvous Server. The Device GUID
is included to help a REST server create a token. It is not otherwise needed.

The “g2” variable is the GUID of the Device. This may be used as an index by the
Rendezvous Service to look up information associated with the Device.

The “eA” variable contains signature related information, as described in
section [§](../protocol-data-types/#signatures).

### TO1.HelloSDOAck, Type 31

From Rendezvous Server to Device TEE

Message Format:

```json
{
    "n4": Nonce,
    "eB": SigInfo
}
```

Message Meaning:

Sets up Device TEE for next message.

The “n4” tag contains a nonce to use as a guarantee of signature freshness in
the TO1.ProveTOSDO.

The “eB” variable contains signature related information, as described in
in section [§](../protocol-data-types/#signatures).

### TO1.ProveToSDO, Type 32

From Device TEE to Rendezvous Server:

Message Format:

```json
{
    "bo": {
	"ai": AppId,
	"n4": Nonce,
	"g2": GUID,
    },
    "pk": PublicKey, # Intel<sup>®</sup> EPID key for Intel<sup>®</sup> EPID device attestation; PKNull if ECDSA
    "sg": Signature
}
```

Message Meaning:

Proves validity of device identity to the Rendezvous Server for the Device
seeking its owner, and indicates its GUID, “g2”.

AppID provides evidence of the TEE application that is running, which indicates
that the behavior may be trusted. It is verified using information from the TEE
author (For example, Intel).

Nonce4 proves that the signature was just computed, and not a reply (signature
‘freshness’ test).

The field “pk” shall contain the Intel<sup>®</sup> EPID public key in case Intel<sup>®</sup> EPID device
attestation. It shall be PKNull in case of ECDSA device attestation.

If the device signature cannot be verified, or fails to verify, the connection
is terminated with an error message (see section [§](#error-type-255)). When the device
attestation method is Intel<sup>®</sup> EPID, the signature is checked with Intel<sup>®</sup> EPID group
keys (For example, from Intel). In the case of ECDSA, the leaf certificate in the
device certificate chain contained in the Ownership Voucher is used to verify
the signature.

### TO1.SDORedirect, Type 33

From Rendezvous Server to Device TEE:

Message Format:

The exact value of: TO0.OwnerSign.to1d, being:

```json
{
    "bo": {
	"i1": IPAddress, # IP address where we are waiting
	"dns1": String, # DNS address where we are waiting (alternative)
	"port1": UInt16, # TCP/UDP port number
	"to0dh": Hash # Hash(to0d object, brace to brace)
    },
    "pk": PKNull,
    "sg": Signature # signed with “Owner key” that Device will get in TO2
}
```

Message Meaning:

Indicates to the Device TEE that a new Owner is indeed waiting for it, and may
be found by connecting to the given DNS name or IP address. If only an IP
address is needed, the DNS String can be empty (zero length). If a DNS name is
present, the DNS lookup is performed first, and all resolved IP addresses are
tried before the given IP address is tried. If the given IP address was one of
the IP addresses returned by DNS, it does not have to be tried separately (once
is enough).

!!! Note
    This message is bit-for-bit identical to TO1.OwnerSign.to1d.

## Transfer Ownership Protocol 2

The Device communicates with the Owner Client based on the values in the
[TO1.SDORedirect] message.

The TO2 Protocol is the most complicated of the protocols in Secure Device Onboard, because it has several steps that are not present in other protocols:

-   Establishes trust in both directions: The Device uses its device attestation
    key and the Owner uses the Ownership Voucher.

-   Creates an encrypted channel, based on the above trust, using a supported
    key exchange mechanism.

-   Exchanges device service info for owner service info.

-   The Owner replaces all Secure Device Onboard credentials in the Device
    (this does not include the Device’s hardware root of trust); the Device
    gives the Owner an HMAC that allows it to generate a replacement Ownership
    Voucher. The Owner can use this new Ownership Voucher in future Secure Device Onboard
    transactions (For example, to resell the Device).

In addition, in all these operations, all unbounded data items are divided
across multiple messages, to limit the size of an HTTP transaction that the
Device is required to process (ideally, each message fits into a single packet;
this is not guaranteed at present). This causes several loops in the protocol:

-   The Ownership Voucher is transmitted header first, then entry by entry in
    successive messages.

-   The service info (in each direction) is transmitted in as many messages as
    necessary to keep the message size to a single packet. A constrained device
    may assume that the connection MTU size is 1500 bytes. The Owner should try
    to keep the size of each service info message down to less than 1300 bytes,
    to allow constrained device protocols to operate correctly.

The ServiceInfo exchange in Secure Device Onboard allows the cooperating client entities on
the Device and Owner to negotiate their own “protocol” for setting up the
Device. The names and meanings of key value pairs is generally up to the Device
and Owner, but specific (useful) values are given in section [§](#to0acceptowner-type-25).

<figure id="fig-TO2Protocol">
<!--    <img src="../img/4ebd125c046e25cefce966baf8b60bba.emf" alt="TO2 Protocol"/> -->
    <img src="../img/to2-protocol-bounce.png" alt="TO2 Protocol">
    <figcaption>Transfer Ownership Protocol 2 (TO2)</figcaption>
</figure>

### Limitation of Round Trip Times

The implementation shall complete the Transfer Ownership Protocol 2 in no more than 1,000,000 round trip times, overall.  Owner and Device implementations should not request more iterations than this.

### TO2.HelloDevice, Type 40

From Device TEE to New Owner Client

Message Format:

```json
{
    "g2": GUID,
    "n5": Nonce,
    "pe": Uint8, # Public key encoding
    "kx": String, # key exchange suite name
    "cs": String, # Ciphersuite name
    "eA": SigInfo # Device attestation signature info
}
```

Message Meaning:

Sets up new owner for proof of ownership.

The “pe” field indicates the preferred public key encoding.

The “kx” and “cs” fields indicate the key exchange protocol and cipher suite to
use. Because we assume the Device may be constrained, it gets to choose these
values; the Owner side must support all choices that a Device can make.

The value for “kx” is given in section [§](../protocol-description/#key-exchange-in-the-to2-protocol), either:

-   Secure Device Onboard 1.0 & Secure Device Onboard 1.1 protocol spec: DHKEXid14, ASYMKEX, or ECDH

-   Future crypto: DHKEXid15, ASYMKEX3072, or ECDH384

The cipher suite “bo.cs” is as given [here](../data-transmission-persistence/#encrypted-message-body) in Table 2. Cipher Suite Names and*
Meanings.

Other key exchange protocols or cipher suites may be supported in the future.

The “eA” tag starts the Device’ signature process.

### TO2.ProveOPHdr, Type 41

From New Owner Client to Device TEE:

Message Format:

```json
{ # Signature of OP Owner Key
    bo: {
	"sz": UInt8, # Ownership Voucher “sz” tag
	"oh": {#Ownership Voucher Hdr
	    "pv": UInt16,
	    "pe": UInt8,
	    "r": Rendezvous,
	    "g": GUID,
	    "d": String,
	    "pk": PublicKey,
	    "hdc": Hash # Absent if using Intel EPID
	},
	"hmac":Hash, # Ownership Voucher “hmac” tag
	"n5": Nonce, # n5 from TO2.HelloDevice
	"n6": Nonce, # used below in TO2.ProveDevice and TO2.Done
	"eB": SigInfo, # Device attestation signature info
	"xA": KeyExchange # Key exchange first step
    },
    "pk": PublicKey, # owner public key, may not be PKNull
    "sg": Signature
}
```

Message Meaning:

This message serves several purposes:

-   The Owner begins sending the Ownership Voucher to the device (only the
    header is in this message).

-   The Owner signs the message with the Owner key (the last key in the
    Ownership Voucher), allowing the Device to verify (later on) that the Owner
    controls this private key.

-   The Owner starts the key exchange protocol by sending the initial key
    exchange parameter xA (For example, in Diffie Hellman, the parameter ‘A’) to the
    Device.

The Ownership Voucher’s header is sent in the “oh” and “hmac” tags. The “sz” tag
gives the number of Ownership Voucher Entries. The entries will be sent in
subsequent messages. It is legal for the “sz” tag to have a value of zero (0),
but this is only useful in re-manufacturing situations since the Rendezvous
Service cannot verify (or accept) these Ownership Proxies.

The “hmac” tag is a HMAC-SHA256 (Secure Device Onboard 1.0 and Secure Device Onboard 1.1 protocol spec)
or HMAC-SHA384 (future crypto) over the “oh” tag. The HMAC key is the one that
was created in the Device during the DI Protocol (or stored in the Device, if an
alternate mechanism is used to initialize the Device). The Device re-computes
the HMAC value against the received contents of the “oh” tag using this stored
secret, and verifies that the “hmac” tag has the same value. This ensures that
the Device itself has not been reinitialized since it was originally programmed
during manufacturing.

For ECDSA device attestation method, the New Owner Client includes the hash of
device certificate chain from Ownership Voucher (OP.oh.hdc) in the
TO2.ProveOPHdr message (as TO2.ProveOPHdr.bo.oh.hdc) for the device to verify
the HMAC. The device temporarily saves the oh.hdc on receiving the message. When
the device computes the new HMAC based on the fields received inTO2.SetupDevice
message, it uses the value of TO2.ProveOPHdr.bo.oh.hdc that was previously
saved. The new HMAC is returned to the New Owner Client as part of TO2.Done
message

The public key “pk” is the Owner Key. This key, which verifies the message
signature (“sg”), must be compared with the public key in the last Ownership
Voucher Entry when it is received (later in the sequence of this protocol).[^22]

[^22]: Note that bo.oh.pk is the initial owner public key from the Ownership
Proxy Header, and should not be confused with bo.pk.

This key must also be able to verify the signature of the TO1.SDORedirect
message. The Device must store the TO1.SDORedirect message (or its hash) until
the TO2.ProveOPHdr message is received. At this time, the Device can verify the
TO1.SDORedirect signature with the give Owner key in TO2.ProveOPHdr.pk. If the
TO1.SDORedirect signature does not verify, the Device must assume that a man in
the middle is monitoring its traffic, and fail immediately with an error code
message.

The bo.eB field continues the Device’ signing process.

The bo.xA field begins the key exchange protocol. See section [§](../protocol-description/#key-exchange-in-the-to2-protocol)  for more
details on key exchange. The key exchange is finished in the TO2.ProveDevice message (section [§](../detailed-protocol-description/#to2provedevice-type-44)).


The verification of this message is critical, but may be a little hard to
understand. The Device initially verifies this message’s signature “sg” using
the supplied key “pk”, then saves a copy of this key (for memory reasons, the
Device may save a SHA hash of the key). As the Ownership Voucher entries are
transmitted in successive TO2.GetOPNextEntry messages, the Device can verify
them using the signature chain embedded in the Ownership Voucher, from header to
entry 1 to entry 2, and so on. The last such entry signs bo.pk, which is called
the “owner key”. Now the Device must verify that the Owner can sign with this
bo.pk public key’s corresponding private key. But if this bo.pk matches the
TO2.ProveOPHdr.pk, then the signature verification at the start has verified
exactly this. The following diagram illustrates the process, using only the
signature chain, for an Ownership Voucher 3 entries:

<figure id="fig-OVVerification">
<!--    <img src="../img/5e53c6f04dfa19326445cc67a9c3ffac.emf" alt="OV Verification"/> -->
    <img src="../img/fig-ov-verification.png" alt="OV Verification">
    <figcaption>Verification of Ownership Voucher by Device</figcaption>
</figure>

### TO2.GetOPNextEntry, Type 42

From Device TEE to New Owner Client:

Message Format:

```json
{
    "enn": UInt8
}
```

Message Meaning:

Acknowledges the previous message and requests the next Ownership Voucher Entry.

### TO2.OPNextEntry, Type 43

From New Owner Client to Device TEE

Message Format:

```json
{
    "enn":UInt8,
    "eni":{
	"bo":{
	    "hp": Hash,
	    "hc": Hash,
	    "pk": PublicKey
	},
	"pk": PKNull,
	"sg": Signature
    }
}
```

Message Meaning:

Transmits the requested Ownership Voucher entry from the New Owner to the Device
TEE. The value of tag “enn” matches the value of TO2.GetOPNextEntry.enn.

If enn == TO2.ProveOPHdr.bo.sz-1, then the next state is TO2.ProveDevice.
Otherwise the next state is TO2.GetOPNextEntry.

The Device TEE verifies the ownership voucher entries incrementally as follows:

-   Variables:

    -   hp – hash of previous entry. The hash only covers the “bo” in the previous
    entry with the “bo:” tag but includes the enclosing braces . For the first
    entry, the hash is SHA [TO2.ProveOPHdr.bo.oh\|\|TO2.ProveOpHdr.bo.hmac].

    -   pk – public key signed in previous entry (initialize with
    TO2.ProveOPHdr.bo.oh.pk)

    -   hc – hash of GUID and DeviceInfo, compute from TO.OwnerSign.bo as:
    SHA[TO2.ProveOPHdr.bo.oh.g\|\|TO2.ProveOPHdr.bo.oh.d]

    -   Use SHA256 (Secure Device Onboard 1.0 and Secure Device Onboard 1.1 protocol spec) or SHA384 (future
    crypto).

    -   Pad the hash text on the right with zeros to match the hash length.

-   For each entry:

    -   Verify signature TO2.OPNextEntry.eni.sg using variable pk

    -   Verify variable hc matches TO2.OPNextEntry.eni.bo.hc

    -   Verify hp matches TO2.OpNextEntry.eni.bo.hp

    -   Update variable pk TO2.OPNextEntry.eni.bo.pk

    -   Update variable hp SHA [TO2.OpNextEntry.eni.bo]

    -   (SHA256 for Secure Device Onboard1.0 and Secure Device Onboard 1.1 protocol spec and SHA384 for
        future crypto)

-   If enn == TO2.ProveOpHdr.bo.sz-1 then verify TO2.ProveOPHdr.pk ==
    TO2.OpNextEntry.eni.bo.pk

### TO2.ProveDevice, Type 44

From Device TEE to New Owner Client

Message Format:

```json
{ # Signature
    "bo": {
	"ai": AppId, # proves App provenance within TEE
	"n6: Nonce, # proves signature freshness
	"n7: Nonce, # used in TO2.SetupDevice
	"g2": GUID, # proves the GUID matches with g2 in TO2.HelloDevice and TO2.Done2
	"nn": UInt8, # number of device service info messages to come
	"xB": DHKeyExchange # Key Exchange, 2nd Step
    },
    "pk": PublicKey, # Intel<sup>®</sup> EPID key for Intel<sup>®</sup> EPID device attestation; PKNull if ECDSA
    "sg": Signature
}
```

Message Meaning:

Proves the provenance of the device to the new owner, using the device
attestation signature (“sg”) based on the challenge (nonce) “n6”.

If the signature cannot be verified, or fails to verify, the connection is
terminated with an error message (section [§](#error-type-255)).

Completes the key exchange, by sending “xB”.

Sends “n7” for later use.

Sends “nn” to indicate number of TO2.GetNextDeviceServiceInfo messages.

The field “pk” shall contain the Intel<sup>®</sup> EPID public key in case of Intel<sup>®</sup> EPID
device attestation. It shall be PKNull in case of ECDSA device attestation.

!!! Note
    Subsequent message bodies are HMac’d and Encrypted. See section ‎[§](../protocol-description/#key-exchange-in-the-to2-protocol) for more details on key exchange.

### TO2.GetNextDeviceServiceInfo, Type 45

From New Owner Client to Device TEE

Message Format - after decryption and verification of HMAC:

```json
{
    "nn": UInt8, #Index of device service info message expected
    "psi": String # extra for this version of protocol only
}
```

Message Meaning:

Acknowledges the TO2.ProveDevice message.

Requests the next Device Service Info message.

-   “nn” is the index of the next device service info expected, starting at
    zero.

-   “psi” is an optional string that may be used to inform the Device before it
    generates the Device Service Info. The value of “psi” is only significant
    when “nn” == 0, and otherwise “psi” must be the empty string. The value of
    “psi” may be used when the Device Service Info is dependent on information
    from the Owner. For example, if the Owner needs the Device to allocate 4 key
    pairs, the “psi” variable may be used to transmit the number “4” as a
    string.[^23]

    [^23]: This facility is intended to work around a problem that will be
    solved by multiple “rounds” of Service Info in a future release of Secure Device Onboard.

### TO2.NextDeviceServiceInfo, Type 46

From Device TEE to New Owner Client

Message Format - after decryption and verification of HMAC:

```json
{
    "nn": UInt8, # index of this message, from zero upwards.
    "dsi": ServiceInfo # service info entries to add or append to previous ones.
}
```

Message Meaning:

Sends as many Device to Owner ServiceInfo entries as will conveniently fit into
a message, based on protocol and Device constraints.

When entries are received with the same name, the second and subsequent entries
are concatenated onto the end of the previous entry. For example, a 3000 byte
entry “cert” might be sent in 3 successive TO2.DeviceServiceInfo messages, one
containing a “cert” entry with the first 1000 bytes, the second containing a
“cert” entry with the second 1000 bytes, and the third containing a “cert” entry
with the last 1000 bytes.

This facility is intended to allow ServiceInfo entries to be large, but still
fit into constrained message sizes.

If “nn” == TO2.ProveDevice.bo.nn-1 then the next message is TO2.SetupDevice.

Otherwise, the next message is TO2.GetNextDeviceServiceInfo.

!!! Note
    ServiceInfo messages might need to be converted to use &bsol;uFFFF syntax to
    avoid internal characters.

### TO2.SetupDevice, Type 47

From New Owner Client to Device TEE

Message Format - after decryption and verification of HMAC:

```json
{
    "osinn":UInt8, # number of service info messages to come
    "noh":{ # update to ownership voucher header for resale.
	"bo":{
	    "r3": Rendezvous, # replaces stored Rendevous
	    "g3": GUID, # replaces stored GUID
	    "n7": Nonce # proves freshness of signature
	},
	"pk": PublicKey, # Owner2 key (replaces Manufacturer’s key).
	"sg": Signature # Proof of Owner2 key.
    }
}
```

Message Meaning:

This message effects ownership transfer, causing the credentials previously used
to take over the device to be replaced. If we recall the credentials programmed
in the DI protocol, these are now updated based on the new credentials
downloaded from the new Owner.


<table>
    <caption>Table 2 - Message Meaning Transformation - DI to T02</caption>
    <thead>
        <tr>
            <th>Old Entry (section ‎5.3.1)</th>
            <th>New Value (Typical use case)</th>
            <th>New Value (Credential Reuse)</th>
            <th>Comments</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>DI.SetCredentials.oh.**pv**</td>
            <td>unchanged</td>
            <td>unchanged</td>
            <td>Protocol version (fixed)</td>
        </tr>
        <tr>
            <td>DI.SetCredentials.oh.**pe**</td>
            <td>unchanged</td>
            <td>unchanged</td>
            <td>Public key encoding (fixed)</td>
        </tr>
        <tr>
            <td>DI.SetCredentials.oh.**r**</td>
            <td>TO2.SetupDevice.noh.bo.**r3**</td>
            <td>unchanged</td>
            <td></td>
        </tr>
        <tr>
            <td>DI.SetCredentials.oh.**g**</td>
            <td>TO2.SetupDevice.noh.bo.**g3**</td>
            <td>unchanged</td>
            <td></td>
        </tr>
        <tr>
            <td>DI.SetCredentials.oh.**d**</td>
            <td>Unchanged</td>
            <td>unchanged</td>
            <td>Manufacturer’s info</td>
        </tr>
        <tr>
            <td>DI.SetCredentials.oh.**pk**</td>
            <td>TO2.SetupDevice.noh.**pk**</td>
            <td>unchanged</td>
            <td>noh.pk is Owner2 key</td>
        </tr>
    </tbody>
</table>

The “osinn” field gives the number of owner service info messages that will be
transmitted to the Device. The Owner should try to keep the size of each service
info message down to less than 1300 bytes, to allow constrained device protocols
to operate correctly.

!!! Note
    See Section [§](../resale-protocol/#resale-protocol) for additional information on Resale protocol, and section [§](../credential-reuse-protocol/#credential-reuse-protocol) for additional information on Credential Reuse protocol.

### TO2.GetNextOwnerServiceInfo, Type 48

**From Device TEE to New Owner Client**

**Message format, after decryption and verification of HMAC:**

```json
{
    "nn":Uint8
}
```

**Message Meaning:**

Requests next Owner Service Info message.

### TO2.OwnerServiceInfo, Type 49

**From New Owner Client to Device TEE**

**Message Format - after decryption and verification of HMAC:**

```json
{
    "nn": UInt8, # index of this message, from zero upwards
    "sv": ServiceInfo
}
```

**Message Meaning:**

Contains ServiceInfo key value pairs, transmitted from Owner to Device. The
Owner should send only enough pairs to keep within the likely constraints of the
Device and the protocol. In particular, no more than 1300 bytes of ServiceInfo
should be sent in a single message.

ServiceInfo entries that are duplicated in subsequent messages are *appended* to
the same named entries at the destination. This makes it possible to send an
arbitrary sized ServiceInfo message.

See section [§](../protocol-description/#management-agent-service-interactions-using-serviceinfo) for information on service info.

If TO2.OwnerServiceInfo.nn == TO2.SetupDevice.osinn-1 then the next state is
TO2.Done.

Otherwise the next state is TO2.GetNextOwnerServiceInfo.

### TO2.Done, Type 50

**From Device TEE to New Owner Client:**

**Message Format - after decryption and verification of HMAC:**
      
```json
{
    "hmac:": Hash,
    "n6:": Nonce # Nonce generated by New Owner Client
                 # and sent to Device TEE in Msg TO2.ProveOPHdr
}
```

**Message Meaning:**

Indicates successful completion of the Transfer of Ownership.

The Client and Owner software now transitions to performing the requested
actions between Device and Owner. For example, the Client can activate the
scripting implicit in the ServiceInfo data structure received from the
Rendezvous Server.

The Device TEE discards and regenerates the secret from the DI Protocol. It then
generates an “hmac” field equivalent to the one in the
[DI.SetHMac] message (see section [§](#disethmac-type-12)).

The Owner may use this information to construct a new Ownership Voucher based on
the Owner2 key and the new information configured into the Device in the
TO2.SetupDevice message. This information permits the Owner to effect a new
transfer of ownership by re-enabling the Secure Device Onboard software on the Device.

If the Device does not support resale (see section [§](../resale-protocol/#resale-protocol)), and wishes to so
inform the Owner, the HMAC is returned with zero length. It is legal for the
Device to generate a valid HMAC but refuse to support resale at a later time. In
this case, it is highly recommended that an out-of-band mechanism be provided to
let the Owner know that the resale protocols will not work.

If the Device supports Credential Reuse protocol and all the conditions for
Credential Reuse are satisfied in TO2.SetupDevice, then a special value of HMAC
with length 1 is returned (see section ‎[§](../credential-reuse-protocol/#credential-reuse-protocol)).

### TO2.Done2, Type 51

**From New Owner Client to Device TEE:**

**Message Format - after decryption and verification of HMAC:**

```json
{
    "n7:": Nonce # Nonce generated by Device TEE and send to Owner in TO2.ProveDevice
}
```

**Message Meaning:**

This message provides an opportunity for a final ACK after the Owner has invoked
the System Info block to establish agent-to-server communications between the
Device and its final Owner.

When possible, the TO2.Done2 should be delayed until the Device has established
agent-to-server communications, allowing an Secure Device Onboard error to occur when such
communications fail.

On some constrained devices, Secure Device Onboard software might not be able to run after
the agent-to-server communications are set up. On these systems, this ACK can
happen right after the TO2.Done message. Such systems cannot recover from a
failure that appears after Secure Device Onboard has finished, but that prevents
agent-to-server communications from being established.

Examples of systems that cannot generate a response after agent-to-server
communications are working include:

-   Constrained systems that don’t have enough resources to run both Secure Device Onboard
    and the agent-to-server subsystems.

-   Systems that require a reboot to complete agent-to-server setup.

## After Transfer Ownership Protocol Success

-   The New Owner Client transfers device information to the manager server.

-   The Device TEE indicates to its user-space handler to invoke the Device to
    Manager Agent for the manager service. This Device to Manager Agent should
    be given all the information that the TEE has now collected.

-   The Device TEE transitions to the IDLE state.

-   The new Owner has changed all credentials in the device, except the Device
    hardware root of trust (and some manufacturer-specific credentials) and has
    sufficient information to construct an Ownership Voucher with zero entries.

