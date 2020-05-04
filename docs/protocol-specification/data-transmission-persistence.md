# Data Transmission & Persistence

## Message Format

All data is transmitted using Messages. Data may also be persisted or
interchanged using messages, although a more compact form of the message might
be used for long-term storage; for example, a persisted message might be
compressed or re-encoded in a system-dependent binary format.

A message has 3 or 4 elements:

-   A length

-   A numeric type, see section [§](../protocol-data-types#numeric-types)

-   Protocol version: the version of this protocol, expressed as a single
    integer being the major version \* 100 + the minor version. For example,
    version 3.25 is represented as: 325.

-   When present, a body. The body is a JavaScript\* Object Notation (JSON)
    object, transmitted using the encoding of section  [§](../protocol-description#json-distinguished-encoding): JSON\* Distinguished
    Encoding. The format of the body is determined by the message type.

The other message elements are transmitted differently, depending on the
protocol in use:

-   When Secure Device Onboard protocols are transmitted using a reliable stream protocol
    when Secure Device Onboard objects are persisted offline or using TCP, TLS, Bluetooth<sup>®</sup>
    technology, or the message is formatted as a specially-encoded JSON*
    sequence, containing all the message elements within it.

-   When Secure Device Onboard protocols are transmitted using RESTful protocols, length,
    numeric type, and protocol version are encoded into the REST headers. The
    body is transmitted as a REST payload. See section [§](#transmission-of-messages-over-a-restful-protocol): Transmission of
    Messages over a RESTful Protocol.

## Transmission of Messages over a Stream Protocol and Persisted Messages

In some cases, messages are transmitted over a stream or datagram protocol. This
is a protocol that reliably transmits a stream of data with no external or
out-of-band information. In this case, all message data must be encapsulated in
a single JSON* encoding.

Persisted Messages also use this encapsulation, please see section [§](#persistence-of-messages): Persistence of Messages* for more information.

In the stream encapsulation of Secure Device Onboard, messages contain their own header,
which is formed by encapsulating the message in a JSON* array with 3 or 4
elements:

[length, messageType, protocolVersion]

or

[length, messageType, protocolVersion, body]

Where:

-   Length appears in hex with *exactly 6 hex digits*, this is an exception
    to the normal encoding rules. The length includes all the characters in the
    message, including the length field itself. For example: ["000d",15,4] shows
    a message with 13 characters (including square brackets, with no body), and
    has a length of 0xd = 13. Note that the stream reader can determine the
    length of the message by reading the first 8 bytes, verifying that:

-   bytes 0-1 are open-square bracket and double-quote ([")

-   bytes 2-5 are hexadecimal values in the set: [0-9a-fA-F]

-   bytes 6-7 are double-quote and comma (,)

-   Then the message length in bytes is the hexadecimal number formed by bytes
    2-5. The rest of the message can be read as the next message-length-8 bytes
    (the first 8 bytes have already been read, above).

-   Type: appears as a UInt8.

-   ProtocolVersion: appears as a UInt16.

-   Body: when present, the body is a JSON* object (including the brace
    brackets). The format and presence of the body is determined by the
    messagetype field.

When a stream protocol is used as the transport for the JSON* messages, the
protocol proceeds as follows:

-   The Device always calls out as the stream client.

-   The Rendezvous Server always acts as the stream server.

-   The Owner Client acts as stream client for Transfer Ownership Protocol 0  
    TO0—interacting with the Rendezvous Server) and as stream server for
    Transfer Ownership Protocol 2 (TO2—interacting with the device).

-   Server port assignment is currently defined as: TO0: 8040; TO1: 8041; TO2: 8042

-   Note that this does NOT affect port assignment when using RESTful protocols.

-   Messages are sent in the stream verbatim, without a separate messaging
    layer. Note that the message format is specifically designed with a fixed
    length field at the start (["000d",…) to allow the reader to read the length
    prefix of the message as a exactly 8 bytes and use this value to read the
    rest of the message exactly (*length* – 8 bytes).

-   The stream is kept open until the last message has been transmitted, then
    dropped using the normal stream close (For example, TCP FIN).

### Maintenance of Stream Connection

When the client uses a stream connection, the entire connection proceeds across
a single stream connection. In the case of a TCP stream, the client and server
must configure their TCP implementation to send “keep-alives” frequently enough
to keep the connection alive for the entire protocol transaction, *including*
all stateful routers and firewalls that might be in the connection path. This is
particularly an issue if either the client or the server takes a long delay to
send some messages.

## Transmission of Messages over a RESTful Protocol

When a REST protocol is used as transport for JSON* messages, the protocol
proceeds as follows:

-   The REST client always uses an HTTP POST. The content type is
    application/json.

-   The REST server listens on a standard port[^21] for the transport protocol.  
    (HTTP: TCP/80, TLS: TCP/443)

[^21]: When testing on a single machine, it is useful to use non-standard
    ports, such as the TCP stream ports (8040, 8041, 8042). However, in service
    and by default, standard ports should be used.

-   A Device which supports both TLS- and TCP-based REST will choose TLS for
    port 443 and TCP for all other ports.

-   If the port is not specified in RendezvousInfo, then for each IP address,
    TLS is tried first on port 443, then TCP is tried on port 80

-   It is possible for an Secure Device Onboard Device to support only TCP; an Secure Device Onboard
    Owner must support both TLS and TCP to support all possible Secure Device Onboard
    Devices. However it is recommended that Secure Device Onboard Devices support TLS where
    feasible.

-   Each REST transaction corresponds to a pair of JSON* messages.

-   The first message body is delivered in the POST body.

-   The second message is delivered as the entire POST response.

-   The length of the message is derived from the Content-Length field.

-   The URL for the message is of the form:

-   /mp/protocolversion/msg/msgnum  
    Where “/mp/” is verbatim; protocolversion is the protocol version number,
    expressed as a 3 digit decimal number; “/msg/” is verbatim, and msgnum is
    the message type (For example, from the table in section [§](#message-types)).

-   On first message, the REST server allocates a token, which must be
    maintained by the REST client for the duration of the connection.

-   The token is transmitted in the REST “Authorization” header.

-   The form of the token is implementation-specific. The simplest token is just
    a random number chosen to be unique from other tokens. A JSON* Web Token
    (JWT) might also be used.

-   The purpose of the token is to link REST calls to their protocol context
    within the JSON\* message stream defined by the Secure Device Onboard
    Protocols. For example, a Java\* implementation of Secure Device Onboard protocols might
    use a Java* object to store connection state. A new REST call can find this
    stored state by looking it up using the token as a key.

The Secure Device Onboard implementation has some latitude in both the form of the
authorization token and how this token gets allocated for Secure Device Onboard protocols.
In the typical (recommended) case, there is no initial authorization:

-   The initial REST request from the Device has an empty Authorization header
    or no such header. Secure Device Onboard protocols perform their own authorization
    within the message layer.

-   The Rendezvous Server detects such a header as a request for a new
    connection, and allocates a new token and associates it with the protocol
    context.

-   The Device saves the token and uses it on subsequent requests within the
    protocol, but not across protocols. An example is when TO1 uses one token,
    and TO2 uses a different token.

-   The Rendezvous Server uses the token to look up the protocol context so that
    each subsequent message is processed correctly.

If the Rendezvous Server wishes to obtain a token using specific REST
credentials, these must be programmed into the device, then transmitted with or
before the first Secure Device Onboard REST request. How this might be
done is outside the scope of this document.

### Maintenance of REST Connections

When transmitting messages across REST transactions, the client and server must
take into account the possibility that the underlying network connection may
time out between REST transactions. This is a problem if the time between a REST
message and its response (that is, a POST and the POST response) is long or if the
time between messages is long.

In general, each Secure Device Onboard protocol may send RESTful transactions across a
single TCP stream (or SSL stream for HTTPS). We require that the TCP server side
(the Owner or the Rendezvous Service) either respond to messages within one or
two seconds, or generate TCP keep-alives sufficient to keep the connection open.

In the case of the TO1 and TO2 Protocols, the client is the Device, which might
be running on a limited processor. In this case, some of cryptographic
operations may take long enough for the underlying TCP connection to timeout
between messages. The client must be robust in its ability to restore TCP / TLS
connections for each RESTful transaction.

It is legal for the client to open a new TCP connection for each RESTful
transaction, although it is recommended that the connection be used for multiple
transactions where possible.

## Persistence of Messages

A message may be persisted by storing it in a permanent medium. If necessary,
multiple messages may be persisted in the same medium, one after the other.
Since a message is a JSON\* statement, it is legal JSON\* to store messages in a
JSON\* object or JSON\* sequence.

As the name “message” suggests, most messages are intended for ephemeral
transmission only. Messages intended for persistence are defined to help in
building tools for the following situations:

-   Defining, storing, and extending the Ownership Voucher.

-   Defining, signing, and storing the Ownership and Manufacturing Credentials.

-   Storing and exchange of public keys during extension of the Ownership
    Voucher.

## Encrypted Message Body

Transfer Ownership 2 Protocol (TO2) includes a key exchange (for more
information, see section [§](../protocol-description/#key-exchange-in-the-to2-protocol)), which
generates a session encryption key (SEK) and a session verification key (SVK).
Subsequent message bodies in this protocol are protected by HMAC\[SVK] and
subsequently encrypted using the session key (Cipher\[SEK]). An encrypted message
has the following format:

-   A message header, as per section [§](#message-format).

-   A message body:


<table>
    <caption>Table 1 - Encrypted Message Body Format</caption>
    <thead>
        <tr>
            <th>Code Sample</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
```json
{
    "ct": [
        IVData,		# cipher initialization vector
	    UInt16,		# number of bytes in crypttext
	    ByteArray	# crypttext (encrypted bytes)
    ],
    "hmac": [	# HMAC of message, see below.
        UInt8,  # number of bytes of HMAC
	    ByteArray # HMAC bytes
    ]
}
```
        </tr>
    </tbody>
</table>

For example:

```json
{"ct":[[16, “MTIzNDU2Nzg5MGFiY2RlZg==”],6,"QTA1Njc4"],"hmac":[32,"bbddee …44-b64-chars… 45="]}
```

The HMAC in tag “hmac” should be type HMAC-SHA256 (Secure Device Onboard 1.0 and Secure Device Onboard
1.1 protocol spec) or HMAC-SHA384 (future crypto). The HMAC algorithm must be
the same as specified in the Cipher Suite field of the TO2.HelloDevice message
(TO2.HelloDevice.cs). The HMAC uses SVK (from the key exchange) as the key. The
HMAC covers:

-   the encrypted message body, which is JSON* ASCII data excluding the
    “ct” tag but includes the enclosing brackets of the “ct” field

-   The “ct” tag contains a base64 representation an encryption of the entire JSON*
message being transmitted. The encryption is one of the ciphers described in the
table below.  
  
  
<table>
    <caption>Table 2 - Cipher Suite Names and Meanings</caption>
    <thead>
        <tr>
            <th>
   Cipher Suite Name 
   (see TO2.HelloDevice)
   Secure Device Onboard
   1.0/1.1
   </td>
            <th>
   Initialization Vector
   (IVData.iv in "ct" message header)
   </td>
            <th>
	    Meaning
            </td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
  AES128/CTR/HMAC-SHA256
  (Secure Device Onboard 1.0 and Secure Device Onboard 1.1)
  </td>
            <td>
The IV for AES CTR Mode is 16 bytes long in big-endian byte order, where:

-   The first 12 bytes of IV (nonce) are randomly generated at the
    beginning of a session, independently by both sides.

-   The last 4 bytes of IV (counter) is initialized to 0 at the
    beginning of the session. 

-   The IV value must be maintained with the current session key.
    “Maintain” means that the IV will be changed by the underlying
    encryption mechanism and must be copied back to the current
    session state for future encryption. 

-   For decryption, the IV will come in the header of the received
    message.

The random data source must be a cryptographically strong pseudo
random number generator (CSPRNG) or a true random number generator
(TNRG).
  </td>
            <td>
This is the preferred cipher suite for Secure Device Onboard for
128-bit keys. Other suites are provided for situations where Device
implementations cannot use this suite.
&nbsp;
AES in Counter Mode [6] with 128 bit key using the SEK
from key exchange (‎see section <a href="../protocol-description/index.html#key-exchange-in-the-to2-protocol">§</a>).
  </td>
        </tr>
        <tr>
            <td>
AES128/CBC/HMAC-SHA256
(Secure Device Onboard 1.0 and Secure Device Onboard 1.1)
  </td>
            <td>
IV is 16 bytes containing random data, to use as initialization vector
for CBC mode.  The random data must be freshly generated for every
encrypted message. The random data source must be a cryptographically
strong pseudo random number generator (CSPRNG) or a true random number
generator (TNRG).
            </td>
            <td>
AES in Cipher Block Chaining (CBC) Mode [3] with PKCS7 [17]
padding. The key is the SEK from key exchange (see section <a href="../protocol-description/index.html#key-exchange-in-the-to2-protocol">§</a>).

Implementation notes:

-   Implementation may not return an error that indicates a padding failure.

-   The implementation must only return the decryption error after the
    "expected" processing time for this message. 

It is recognized that the first item is hard to achieve in general,
but Secure Device Onboard risk is low in this area, because any decryption error
will cause the connection to be torn down.
  </td>
        </tr>
        <tr>
            <td>
AES256/CTR/HMAC-SHA384
  </td>
            <td>

The IV for AES CTR Mode is 16 bytes long in big-endian byte order,
where:

-   The first 12 bytes of IV (nonce) are randomly generated at the
    beginning of a session, independently by both sides.  

-   The last 4 bytes of IV (counter) is initialized to 0 at the
    beginning of the session. 

-   The IV value must be maintained with the current session key.
    “Maintain” means that the IV will be changed by the underlying
    encryption mechanism and must be copied back to the current
    session state for future encryption. 

-   For decryption, the IV will come in the header of the received
    message. 

The random data source must be a cryptographically strong pseudo
random number generator (CSPRNG) or a true random number generator
(TNRG). 
            </td>
            <td>
This is the preferred
cipher suite for Secure Device Onboard for 256-bit keys. Other suites are provided for
situations where Device implementations cannot use this suite.
&nbsp;
AES in Counter Mode [6]
with 256 bit key using the SEK from key exchange (see section <a href="../protocol-description/index.html#key-exchange-in-the-to2-protocol">§</a>).
           </td>
        </tr>
        <tr>
            <td>
AES256/CBC/HMAC-SHA384
  (Future crypto)
  </td>
            <td>
IV is 16 bytes containing
random data, to use as initialization vector for CBC mode. 
The random data must be freshly generated for every
encrypted message. The random data source must be cryptographically strong
pseudo random number generator (CSPRNG) or a true random number generator
(TNRG)
            </td>
            <td>
AES-256 in Cipher Block
Chaining (CBC) Mode [15] with PKCS7[16] padding. The key is the SEK from
key exchange (see section <a href="../protocol-description/index.html#key-exchange-in-the-to2-protocol">§</a>).
&nbsp;
Implementation notes:

-   Implementation may not return an error that indicates a padding
    failure.

-   The implementation must only return the decryption error after the
    "expected" processing time for this message. 

It is recognized that the
item is hard to achieve in general, but Secure Device Onboard risk is low in
this area, because any decryption error causes the connection to be torn
down.
  </td>
        </tr>
    </tbody>
</table>

## Message Types

This section defines all persisted and transmitted message types. Each message
is described in greater detail in a later section, but the table here can
provide an overview of the flow of each protocol.

JSON* object tag names are kept small to help constrained implementations. Within
the limits of their brevity, they are chosen for mnemonic value. For example,
the error message “ec” standard for “error code.”

Some values appear in successive versions within the protocol. For example, a
nonce is often used to verify the “freshness” of a signature (that is, that the
signature was performed on demand). Similarly, device GUIDs change, and several
different public keys are used. When this happens, the convention is that the
object tag is a letter describing the object, and a numeral describing the
version. The versions are numbered from 1, based on the Device Initialize
Protocol (DI). Persisted types typically use no numeral.

The following table describes all the messages in the protocol with their
message types and names. The message contents are indicated in column 3, but the
reader is referenced to the detailed section for each message to get a full
description. For example, message comments have been removed.

The REST column of the table indicates the URL to use for this message type,
when the transport is a RESTful protocol (HTTP, HTTPS, and others).


!!! Note
    Messages are not allowed to include whitespace and comments.


### Error Message

The “catch-all” error message is sent whenever processing cannot continue. This
includes protocol errors and any trust or security violations, including:

-   Failure to verify a signature (section ‎[§](../protocol-data-types/#signatures))

-   Revoked EPID Group or signature (SIGRL) (section ‎[§](../protocol-data-types/#intel-enhanced-privacy-id-intel-epid-signatures-overview))

-   Rejection of Application ID or EPID Group Attributes (section [§](../introduction/#intel-enhanced-privacy-id-intel-epid-attributes))

-   Failure to verify the Ownership Voucher against the Device HMAC (section [§](../detailed-protocol-description/#to2proveophdr-type-41))

-   Failure to verify the internal consistency of the Ownership Voucher, or
    failure to verify any of the signatures in the Ownership Voucher against the
    Device Credentials and the Owner key challenge (section [§](../protocol-description/#the-ownership-voucher))

-   Failure to verify the challenge for the Owner key (sections [§](../detailed-protocol-description/#to0ownersign-type-22) & [§](../detailed-protocol-description/#to1sdoredirect-type-33)) or the TO0D signature (section [§](../protocol-description/#the-ownership-voucher))

-   Failure to verify a HMAC or to decrypt a message (section [§](#encrypted-message-body))

-   Failure to interpret the ServiceInfo, or failures internal to the
    ServiceInfo modules (section [§](../protocol-data-types/#serviceinfo-and-management-service-agent-interactions))

-   Failure to verify any nonce with the previously transmitted value (section [§](../detailed-protocol-description/#to0ownersign-type-22) for example)

-   Any resource or communications failure that makes successful onboarding fail

The Secure Device Onboard protocol is always terminated after an error message, and all
Secure Device Onboard error conditions send an error message. However, security errors might
not indicate the exact cause of the problem, if this would cause a security
issue.

The contents of the error message are intended to help diagnose the error. The
“ec” tag is an error code, please see following section, Error Code Values, for
detailed information. The “emsg” tag gives the message ID of the previous
message, making it easier to put the error into context. The “em” tag gives a
string suitable for logging about the error.

The string in the “em” tag must not include security details that are
inappropriate for logging, such as a specific security condition, or any key or
password information.
  

<table>
    <caption>Table 3 - Error Message Content and Meanings</caption>
    <thead>
        <tr>
            <th>Type\#</th>
            <th>Message Type Name</th>
            <th>Message Contents</th>
            <th>From</th>
            <th>To</th>
            <th>REST Transmission</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>255</td>
            <td>Error</td>
            <td>
```json
{ #Error message body
    "ec": Uint16,
    "emsg": UInt8,
    "em": String
}
```
	    </td>
            <td>Any</td>
            <td>Any</td>
            <td>HTTP response with message type 255 when transmitted as HTTP response</td>
        </tr>
    </tbody>
</table>

-   POST /mp/VVV/msg/255, when transmitted as HTTP request

If the problem is found in a HTTP request, the ERROR message is sent as HTTP
response. The body of the response is a Secure Device Onboard/JSON* message with message type 255,
and the “emsg” tag indicates the message type of the HTTP request message. The
flow is as follows:

1.  HTTP request: POST /mp/VVV/msg/X, msg type = X

2.  HTTP response: msg type = ERROR(255), emsg = X

3.  Secure Device Onboard terminates in error on both sides

If the problem is found in a HTTP response, the ERROR message is sent as a new
HTTP request, POST /mp/VVV/msg/255, and the “emsg” tag indicates the message
type of the previous HTTP response message. The authentication token from the
previous HTTP request appears in the HTTP request containing the ERROR message.
Since the ERROR message terminates the Secure Device Onboard protocol, the HTTP response to
an ERROR message is an HTTP empty message (zero length). The flow is as follows:

1.  HTTP request: POST /mp/VVV/msg/Y, msg type = Y (for any message type Y)

2.  HTTP response: msg type = X

3.  HTTP request, POST /mp/VVV/msg/255: msg type = 255, emsg = X

4.  HTTP response: &lt;zero length&gt;

5.  Secure Device Onboard terminates in error on both sides

ERROR messages are never retransmitted, and an ERROR message must never generate
an ERROR message in response.

#### Error Code Values

  
<table>
    <caption>Table 4 - Error Codes</caption>
    <thead>
        <tr>
            <th>Error Code (EC)</th>
            <th>Internal Name</th>
            <th>Generated by Message</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>001</td>
            <td>INVALID_JWT_TOKEN</td>
            <td>DI.SetHMAC TO0.OwnerSign TO1.ProveToSDO TO2.HelloDevice TO2.GetOPNextEntry TO2.ProveDevice TO2.NextDeviceServiceInfo TO2.Done</td>
            <td>JWT token is missing or authorization header value does not start with 'Bearer'. Each token has its own validity period, server rejects expired tokens. Server failed to parse JWT token or JWT signature did not verify correctly. The JWT token refers to the token mentioned in section <a href="../protocol-description/index.html##transmission-of-messages-over-a-restful-protocol">§</a> (which is not required by protocol to be a JWT token). The error message applies to non-JWT tokens, as well.</td>
        </tr>
        <tr>
            <td>002</td>
            <td>INVALID_OWNERSHIP_PROXY</td>
            <td>TO0.OwnerSign</td>
            <td>Ownership Voucher is invalid: One of Ownership Voucher verification checks has failed. Precise information is not returned to the client but saved only in service logs.</td>
        </tr>
        <tr>
            <td>003</td>
            <td>INVALID_OWNER_SIGN_BODY</td>
            <td>TO0.OwnerSign</td>
            <td>Verification of signature of owner message failed. TO0.OwnerSign message is signed by the final owner (using key signed by the last Ownership Voucher entry). This error is returned in case that signature is invalid.</td>
        </tr>
        <tr>
            <td>004</td>
            <td>INVALID_IP_ADDRESS</td>
            <td>TO0.OwnerSign</td>
            <td>IP address is invalid. Bytes that are provided in the request do not represent a valid IPv4/IPv6 address.</td>
        </tr>
        <tr>
            <td>005</td>
            <td>INVALID_GUID</td>
            <td>TO0.OwnerSign</td>
            <td>GUID is invalid. Bytes that are provided in the request do not represent a proper GUID.</td>
        </tr>
        <tr>
            <td>006</td>
            <td>RESOURCE_NOT_FOUND</td>
            <td>TO1.HelloSDO TO2.HelloDevice</td>
            <td>The owner connection info for GUID is not found. TO0 Protocol wasn't properly executed for the specified GUID or information that was stored in database has expired and/or has been removed.</td>
        </tr>
        <tr>
            <td>100</td>
            <td>MESSAGE_BODY_ERROR</td>
            <td>DI.AppStart DI.SetHMAC TO0.Hello TO0.OwnerSign TO1.HelloSDO TO1.ProveToSDO TO2.HelloDevice TO2.GetOPNextEntry TO2.ProveDevice TO2.NextDeviceServiceInfo TO2.GetNextOwnerServiceInfo TO2.Done</td>
            <td>Message Body is structurally unsound: JSON* parse error, or valid JSON*, but is not mapping to the expected Secure Device Onboard type (see section <a href="../protocol-description/index.html#message-types">§</a>)</td>
        </tr>
        <tr>
            <td>101</td>
            <td>INVALID_MESSAGE_ERROR</td>
            <td>TO0.OwnerSign TO1.HelloSDO TO1.ProveToSDO TO2.HelloDevice TO2.GetOPNextEntry TO2.ProveDevice TO2.NextDeviceServiceInfo TO2.GetNextOwnerServiceInfo</td>
            <td>Message structurally sound, but failed validation tests. The nonce didn’t match, signature didn’t verify, hash, or mac didn’t verify, index out of bounds, and others...</td>
        </tr>
        <tr>
            <td>500</td>
            <td>INTERNAL_SERVER_ERROR</td>
            <td>DI.AppStart DI.SetHMAC TO0.Hello TO0.OwnerSign TO1.HelloSDO TO1.ProveToSDO TO2.HelloDevice TO2.GetOPNextEntry TO2.ProveDevice TO2.NextDeviceServiceInfo TO2.GetNextOwnerServiceInfo TO2.Done</td>
            <td>Something went wrong which couldn’t be classified otherwise.&nbsp; (This was chosen to match the HTTP 500 error code.)</td>
        </tr>
    </tbody>
</table>

### Persisted Messages 

  
<table>
    <caption>Table 5 - Persisted Messages Information</caption>
    <thead>
        <tr>
            <th>Type#</th>
            <th>Message Type Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
            <td>PM.CredOwner</td>
            <td>This is the Ownership Credential that is stored in the device TEE during the Device Initialize Protocol (DI), and updated in Transfer Ownership 2 Protocol (TO2).</td>
        </tr>
        <tr>
            <td>2</td>
            <td>PM.CredMfg</td>
            <td>This is the Manufacturing Credential that is stored in the device TEE during the Device Initialize Protocol (DI). It is never changed afterwards.</td>
        </tr>
        <tr>
            <td>3</td>
            <td>PM.OwnershipProxy</td>
            <td>The [Ownership Voucher] is used to convey the trust of the device in the factory to the new owner.</td>
        </tr>
        <tr>
            <td>4</td>
            <td>PM.PublicKey</td>
            <td>This message is used by each successive owner in the supply chain to identify his public key to the previous owner. The previous owner signs the public key (and other information) to extend the ownership voucher.</td>
        </tr>
        <tr>
            <td>5</td>
            <td>PM.ServiceInfo</td>
            <td>This message is used to save the key-value pairs that are sent as part of the TO2.ReceiveDeviceInfo and TO2.SendSetupInfo messages.</td>
        </tr>
        <tr>
            <td>6</td>
            <td>PM.DeviceCredentials</td>
            <td>This message is used to store the device state for implementations that use a filesystem instead of a TEE. It may also be signed or encrypted (sealed) in these implementations, to improve security.</td>
        </tr>
    </tbody>
</table>

### Device Initialize Protocol (DI)

<table>
    <caption>Table 6 - Device Initialize Protocol - Message Information</caption>
    <thead>
        <tr>
            <th>Type\#</th>
            <th>Message Type Name</th>
            <th>Message Contents</th>
            <th>From</th>
            <th>To</th>
            <th>REST Transmission</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>10</td>
            <td>[DI.AppStart]</td>
            <td>
```json
{
    "m": String
}
```
	    </td>
            <td>Dev TEE</td>
            <td>Mfg station</td>
            <td>POST /mp/VVV/msg/10<sup id="fnref:22"><a class="footnote-ref" href="#fn:22">2</a></sup></td>
        </tr>
        <tr>
            <td>11</td>
            <td>[DI.SetCredentials]</td>
            <td>
```json
{
    "oh": { 
	"pv": UInt16, 
	"pe": UInt8, 
	"r": Rendezvous, 
	"g": GUID, 
	"d": String 
	"pk": PublicKey 
	"hdc": Hash #Absent if using Intel EPID
    }
}
```
	    </td>
            <td>Mfg station</td>
            <td>Dev TEE</td>
            <td>(post response, includes authorization token)</td>
        </tr>
        <tr>
            <td>12</td>
            <td>[DI.SetHMAC]</td>
            <td>
```json
{ 
"hmac": Hash
}
```
	    </td>
            <td>Dev TEE</td>
            <td>Mfg station</td>
            <td>POST /mp/VVV/msg/12</td>
        </tr>
        <tr>
            <td>13</td>
            <td>[DI.Done]</td>
            <td>\-no body-</td>
            <td>Mfg station</td>
            <td>Dev TEE</td>
            <td>(post response with token)</td>
        </tr>
    </tbody>
</table>

[^22]: /mp/VVV means ‘/mp/’ followed by the protocol version. For example,
version 1.13 would use /mp/113.

### Transfer Ownership Protocol 0 (TO0) 


<table>
    <caption>Table 7 - Transfer Ownership Protocol 0 – Message Information</caption>
    <thead>
        <tr>
            <th>Type#</th>
            <th>Message Type Name</th>
            <th>Message Contents</th>
            <th>From</th>
            <th>To</th>
            <th>REST Transmission</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>20</td>
            <td>[TO0.Hello]</td>
            <td>
```
-no body-
```
            </td>
            <td>New Owner Client</td>
            <td>Rendezvous Server</td>
            <td>POST /mp/VVV/msg/20</td>
        </tr>
        <tr>
            <td>21</td>
            <td>[TO0.HelloAck]</td>
            <td>
```json
{ 
    "n3": Nonce
}
```
	    </td>
            <td>Rendezvous Server</td>
            <td>New Owner Client</td>
            <td>(post response, includes authorization token)</td>
        </tr>
        <tr>
            <td>22</td>
            <td>[TO0.OwnerSign]</td>
            <td>
```json
{ 
    "to0d": {
	"op": OwnershipProxy, 
	"ws": UInt32, 
	"n3": Nonce
    }, 
    "to1d": { # Sign with Owner key 
	"bo": { 
	    "i1": IPAddress, 
	    "dns1": String, 
	    "port1": UInt16, 
	    "to0dh": Hash
	}, 
	"pk": PKNull, 
	"sg": Signature #Owner key
    }
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Rendezvous Server</td>
            <td>POST /mp/VVV/msg/22</td>
        </tr>
        <tr>
            <td>25</td>
            <td>TO0.AcceptOwner</td>
            <td>
```json
{ 
    "ws": Uint32
}
```
	    </td>
            <td>Rendezvous Server</td>
            <td>New Owner Client</td>
            <td>(post response with token)</td>
        </tr>
    </tbody>
</table>

### Transfer Ownership Protocol 1 (TO1)

  
<table>
    <caption>Table 8 - Transfer Ownership Protocol 1 – Message Information</caption>
    <thead>
        <tr>
            <th>Type\#</th>
            <th>Message Type Name</th>
            <th>Message Contents</th>
            <th>From</th>
            <th>To</th>
            <th>REST Transmission</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>30</td>
            <td>[TO1.HelloSDO]</td>
            <td>
```json
{ 
    "g2": GUID, # device GUID. 
    "eA": SigInfo
}
```
            </td>
            <td>Device TEE</td>
            <td>Rendezvous Server</td>
            <td>POST /mp/VVV/msg/30<br>
	    *GUID added to help token creation*</td>
        </tr>
        <tr>
            <td>31</td>
            <td>[TO1.HelloSDOAck]</td>
            <td>
```json
{ 
    "n4": Nonce, 
    "eB": SigInfo
}
```
	    </td>
            <td>Rendezvous Server</td>
            <td>Device TEE</td>
            <td>(post response, includes authorization token)<br>
            (please see section <a href="../protocol-description/index.html##transmission-of-messages-over-a-restful-protocol">§</a>)</td>
        </tr>
        <tr>
            <td>32</td>
            <td>[TO1.ProveToSDO]</td>
            <td>
```json
{ 
    "bo": { 
	"ai": AppId, 
	"n4": Nonce, 
	"g2": GUID
    }, 
    "pk": PublicKey, #PKNull if ECDSA 
    "sg": Signature
}
```
	    </td>
            <td>Device TEE</td>
            <td>Rendezvous Server</td>
            <td>POST /mp/VVV/msg/32</td>
        </tr>
        <tr>
            <td>33</td>
            <td>[TO1.SDORedirect]</td>
            <td>
```json
{ 
    "bo": { 
	"i1": IPAddress, 
	"dns1": String, 
	"port1": UInt16, 
	"to0dh": Hash
    }, 
    "pk": PKNull, 
    "sg": Signature
}
```
	    </td>
            <td>Rendezvous Server</td>
            <td>Device TEE</td>
            <td>(post response with token)</td>
        </tr>
    </tbody>
</table>

### Transfer Ownership Protocol 2 (TO2) 

  
<table>
    <caption>Table 9 - Transfer Ownership Protocol 2 – Message Information</caption>
    <thead>
        <tr>
            <th>Type#</th>
            <th>Message Type Name</th>
            <th>Message Contents</th>
            <th>From</th>
            <th>To</th>
            <th>REST Transmission</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>40</td>
            <td>[TO2.HelloDevice]</td>
            <td>
```json
{
    "g2": GUID,
    "n5": Nonce,
    "pe": UInt8,
    "kx": String,
    "cs": String,
    "eA": SigInfo
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/40</td>
        </tr>
        <tr>
            <td>41</td>
            <td>[TO2.ProveOpHdr]</td>
            <td>
```json
{# Signature of OP Owner Key
    bo: {
      "sz": UInt8,
      "oh": OwnershipProxyHdr,
      "hmac":Hash,
      "n5": Nonce,
      "n6": Nonce,
      "eB": SigInfo, 
      "xA": KeyExchange
    },
    "pk": PublicKey,
    "sg": Signature
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response, includes authorization token)

<b>Note: </b>
In this message, the pk tag must have the actual Owner key (not
PKNull) to allow Device to verify signature immediately. 

	    </td>
        </tr>
        <tr>
            <td>42</td>
            <td>[TO2.GetOpNextEntry]</td>
            <td>
```json
{
    "enn": UInt8
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/42</td>
        </tr>
        <tr>
            <td>43</td>
            <td>[TO2.OpNextEntry]</td>
            <td>
```json
{ 
    "enn":UInt8,
    "eni":OwnershipProxyEntry
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response with token)</td>
        </tr>



        <tr>
            <td>44</td>
            <td>[TO2.ProveDevice]</td>
            <td>
```json
{
    "bo": {
        "ai": AppId,
        "n6": Nonce,
        "n7": Nonce,
        "g2": GUID,
        "nn": UInt8,
        "xB": DHKeyExchange
    },
    "pk": PublicKey,
    "sg": Signature
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/44

<b>Note: </b> For EPID, "pk" contains EPID group number.  Otherwise, it is PKNull.

	    </td>
        </tr>
        <tr>
            <td>45</td>
            <td>[TO2.GetNext DeviceServiceInfo]</td>
            <td>
```json
{
    "nn": UInt8,
    "psi": String
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response with token)</td>
        </tr>

        <tr>
            <td>46</td>
            <td>[TO2.Next DeviceServiceInfo]</td>
            <td>
```json
{
    "nn”: UInt8, 
    "dsi":ServiceInfo 
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/46</td>
        </tr>
        <tr>
            <td>47</td>
            <td>[TO2.SetupDevice]</td>
            <td>
```json
{
     "osinn":UInt8,
     "noh":{
       "bo":{
            "r3": Rendezvous,
            "g3": GUID, GUID
            "n7": Nonce
        },
        "pk": PublicKey, #Owner2
        "sg": Signature
    }
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response with token)

<b>Note: </b> This is the “owner2” key, the replacement for the manufacturer’s
key in the DI Protocol.  It must not be PKNull.


	    </td>
        </tr>

        <tr>
            <td>48</td>
            <td>[TO2.GetNext OwnerServiceInfo]</td>
            <td>
```json
{
    "nn":Uint8
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/48</td>
        </tr>
        <tr>
            <td>49</td>
            <td>[TO2.Next OwnerServiceInfo]</td>
            <td>
```json
{
    "nn": UInt8, 
    "sv": ServiceInfo
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response with token)</td>
        </tr>

        <tr>
            <td>50</td>
            <td>[TO2.Done]</td>
            <td>
```json
{
    "hmac:": Hash,
    "n6": Nonce
}
```
	    </td>
            <td>Device TEE</td>
            <td>New Owner Client</td>
            <td>POST /mp/VVV/msg/50</td>
        </tr>
        <tr>
            <td>51</td>
            <td>[TO2.Done2]</td>
            <td>
```json
{
    "n7": Nonce
}
```
	    </td>
            <td>New Owner Client</td>
            <td>Device TEE</td>
            <td>(post response with token)</td>
        </tr>

    </tbody>
</table>

