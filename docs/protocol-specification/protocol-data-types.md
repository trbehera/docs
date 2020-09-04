# Protocol Data Types

All data is transmitted in a JavaScript\* Object Notation (JSON) subset, as
described in section [§](../protocol-description/#message-passing-protocol): Message Passing Protocol.

## Base Types

### Numeric Types

Numeric types are transmitted as JSON\* integers. Numbers are represented in JSON\*
using the following representation:


<table>
    <caption>Table 1 - Numeric Types</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Data Size</th>
            <th>JSON* Representation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Uint8</td>
            <td>8 bits</td>
            <td>0..255</td>
        </tr>
        <tr>
            <td>Uint16</td>
            <td>16 bits</td>
            <td>0..65535</td>
        </tr>
        <tr>
            <td>Uint32</td>
            <td>32 bits</td>
            <td>0..4294967303</td>
        </tr>
    </tbody>
</table>

The JSON* representation does not indicate the size of the object. The size
(8-bits, 16-bits, and 32-bits) is given to allow constrained implementations the
ability to select the data size in which to store the result.

Also, there are no negative numbers used in the JSON* subset. The parser does not
have to support them.

All integers will fit in 32 bits of data. However, integers in the range of
0x8000000-0xffffffff may be interpreted as negative numbers on some systems
(For example, Java* integers). The implementations should take care to ensure that the
integers are always treated as positive.

### Boolean

True is represented as the number 1

False is represented as the number 0

JSON\* true and false are not used.

### String Types

Strings are represented as JSON* strings, delimited with double quotation marks
(&quot;). However:

-   Only printable ASCII characters may be in strings (ASCII codes 0x20-0x7e).

-   Non-ASCII characters may be encoded in &bsol;uXXXX format (For example, &bsol;u20ac is the
    euro symbol).

-   To simplify parsing, the follow characters may not be encoded in strings,
    except using the &bsol;uXXXX format:

-   Double quote (", &bsol;u0022)

-   Open and close square brackets ([, &bsol;u005b) and (], &bsol;u005d)

-   Open and close brace brackets ({, &bsol;u007b) and (}, &bsol;u007d)

-   Backslash (&bsol;, &bsol;u005c)

-   Ampersand (&, &bsol;u0026)

-   Backslash escaping is NOT used (For example, a string containing a double quote
    "&bsol;"" must be transmitted as "&bsol;u0022" rather than "&bsol;"").

### ByteArray Types

Byte arrays are represented as a string containing base64 data, padded with
equal signs. The string may contain only the base64 characters, and may not
contain embedded spaces or new lines, regardless of its length. With this
proviso, the base64 encoding follows MIME, as described in
[[RFC2045]]. However, only characters from
the base64 alphabet may be encoded into these strings; whitespace is not
permitted.

For example:

" CgU/GQ==" represents a byte array with 4 bytes, containing 10,5,63,25.

Byte arrays look like JSON\* strings. The context is used to tell them apart.

The length of a ByteArray is not specified in the encoding, but messages always
have a length field sufficient to pre-allocate storage for the ByteArray. This
is not needed for the encoding, but makes constrained implementations easier.
Note that, if the storage for the ByteArray is *n*, the length of the base64
string is *((n+2)/3)\*4*, using integer arithmetic.[^5]

[^5]: 

    To illustrate this, consider the following Python\* script:

    import base64
    s='abcdefghijklmnop'  
    for n in range(0,5):  
        print n, (((n+2)/3)\*4), s[0:n], base64.encodestring(s[0:n]),
 
    Output is:

    0 0  
    1 4 a YQ==  
    2 4 ab YWI=  
    3 4 abc YWJj  
    4 8 abcd YWJjZA==  


To simplify the parser, we specify that ByteArrays always contain encoded data
for every byte (this avoids the need to scan ahead to see how big the input is
compared to the expected buffer size).[^6]

[^6]: It is conceivable to specify a compressed data format, then define that it
is encoded in a ByteArray. We do not currently have such a case in this
document.

## Composite Types

Composite types are combinations or contextual encodings of base types. In the
following table, the type is represented by a JSON\* pseudo-code, where name(type)
is used to indicate the name of the quantity and the JSON\* type.


<table>
    <caption>Table 2 - JSON* Pseudo-Code Types</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Format</th>
            <th>Meaning</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Stream Message<sup id="fnref:7"><a class="footnote-ref" href="#fn:7">3</a></sup></td>
            <td>
```json
[    
    Length((String containing hex),
    Type(integer),
    ProtocolVersion,
    Message Body (Object)
]
```
	    </td>
            <td>See section <a href="../data-transmission-persistence/index.html#data-transmission-persistence">§</a>. The length is stored in hexadecimal format fully specified as 4 hex characters. A message longer than 0xFFFF bytes is stored as a 32-bit hexadecimal number fully specified as 8 hex characters. Implementations should store hex strings using upper case letters. Implementations should parse hex strings whether they have upper case or lower case letters in them. This message header need not be transmitted in some protocols.</td>
        </tr>
        <tr>
            <td>ProtocolVersion</td>
            <td>Integer, represented as: Major * 100 + Minor</td>
            <td>States the protocol version.  For example: version 10.35 is represented as: 1035.</td>
        </tr>
        <tr>
            <td>Hash / HMAC</td>
            <td>
```json
[
    length (UInt8),
    hashtype (UInt8),
    hash[length] (ByteArray)
]
```
	    </td>
            <td>Crypto hash, with length in bytes preceding.  For example, a SHA256 hash looks like:
```json
[32,8,"base64…"]
```
Hashes are computed in accordance with
[[FIPS-180-4]]

A HMAC is encoded as a hash. See the following Wikipedia page for more
information:

```
https://en.wikipedia.org/wiki/Hash-based_message_authentication_code
```
	    </td>
        </tr>
        <tr>
            <td>PKNull</td>
            <td>
```json
[ 0 ]
```
	    </td>
            <td>Encoding of null public key, used as a placeholder when no public key is available.</td>
        </tr>
        <tr>
            <td>PKX509Enc</td>
            <td>
```json
[
    pkBytes, #Bytes in X.509 encoding
    pkx509[pkBytes](ByteArray) #X509 encoding itself 
]
```
	    </td>
            <td>Encoding of Public Key in X.509<sup id="fnref:8"><a class="footnote-ref" href="#fn:8">4</a></sup></td>
        </tr>
        <tr>
            <td>PKRMEEnc</td>
            <td>
```json
[
    modBytes,	       # Number of modulus bytes
    modba(ByteArray),  # Modulus as byte array
    expBytes,	       # Exponent bytes
    expba(ByteArray)   # Exponent as byte array
]
```
	    </td>
            <td>Public Key RSAMODEXP encoding.  Encoding is compatible with Java* BigInteger</td>
        </tr>
        <tr>
            <td>PKEPIDEnc</td>
            <td>
```json
[
   epidGroupNoBytes,	  # size of group number
   epidGroupNo(ByteArray) # group number as byte array
]
```
	    </td>
            <td>Public Key EPID 1.1 encoding.

The group no identifies the specific Intel<sup>®</sup> EPID public key used to
generate the signature. The actual key value is extracted from Intel<sup>®</sup> EPID artifacts are maintained by Intel.
            </td>
        </tr>
        <tr>
            <td>SigInfo</td>
            <td>
```json
[
    sgType(Uint8)
    Length(Uint16),
    Info(ByteArray)
]
```
	    </td>
            <td>
SigInfo is used to encode parameters for the device
attestation signature, This is specifically needed for
Intel<sup>®</sup> EPID revocation and other parameters needed for
Intel<sup>®</sup> EPID signing. sgType is a Signature Type from
section <a href="index.html#device-attestation-signature-and-mechanism">§</a> .
	    </td>
        </tr>
        <tr>
            <td>PublicKey</td>
            <td>
```json
[
    pkType(Uint8),	# Crypto type<sup id="fnref:9"><a class="footnote-ref" href="#fn:9">5</a></sup>
    pkEnc(Uint8),	# Encoding of key<sup id="fnref:12"><a class="footnote-ref" href="#fn:12">8</a></sup>
    PKX509Enc | 	# body
      PKRMEEnc | 
      PKEPIDEnc | 
      PKNull
]
```
	    </td>
            <td>Public key.  Body is one of the above types, depending on the encoding.</td>
        </tr>
        <tr>
            <td>Signature</td>
            <td>
```json
[
    Length(Uint16),
    Signature(ByteArray)
]
```
	    </td>
            <td>The actual signature bits. These come at the end of the signature block.</td>
        </tr>
        <tr>
            <td>SignatureBlock</td>
            <td>
```json
{
     "bo": body(JSON object), # The thing being signed (body)
     "pk": PublicKey,
     "sg": Signature
}
```
	    </td>
            <td></td>
        </tr>
        <tr>
            <td>ManufacturerBlock</td>
            <td>
```json
{# ManufacturerBlock
        "d": String # deviceInfo – manufacturer’s model num
}
```
	    </td>
            <td>Manufacturer block, stored in device</td>
        </tr>
        <tr>
            <td>OwnerBlock</td>
            <td>
```json
{# OwnerBlock
    "pv": UInt16,	# Protocol version
    "pe": UInt8,	# key encoding
    "g": Guid,		# GUID of this device
    "r": RendezvousInfo,  # How to find rendezvous server
    "pkh": Hash		# Hash of mfg's pubkey
}
```
	    </td>
            <td>Ownership block, stored in device</td>
        </tr>
        <tr>
            <td>DeviceCredentials</td>
            <td>
```json
{
    "ST": UInt8 # SDO State
    "Secret": ByteArray # HMAC secret
    "M": {#ManufacturerBlock
        "d": String
    }
    "O": {#OwnerBlock
	"pv": UInt16
	"pe": UInt8
	"r": RendezvousInfo
	"g": Guid
	"pkh": Hash
    }
}
```
	    </td>
            <td>Credentials stored in device</td>
        </tr>
        <tr>
            <td>Nonce</td>
            <td>ByteArray (16 bytes)</td>
            <td>128-bit Random number, intended to be used *only once*.</td>
        </tr>
        <tr>
            <td>GUID</td>
            <td>ByteArray (16 bytes)</td>
            <td>128-bit Random number used for identification. Typically 128 bits.</td>
        </tr>
        <tr>
            <td>IP Address</td>
            <td>
```json
[
    length(Uint8),
    ByteArray
]
```
            </td>
            <td>Length = 4 for IPv4.<br>Length = 16 for IPv6.</td>
        </tr>
        <tr>
            <td>RendezvousInstr</td>
            <td>
```json
[length(Uint8), {ve1:v1,ve2:v2…}]
```
	    </td>
            <td>Length = num of ve:v pairs. VE<sup id="fnref:10"><a class="footnote-ref" href="#fn:10">6</a></sup> stands for Variable Encoding.</td>
        </tr>
        <tr>
            <td>RendezvousInfo</td>
            <td>
```json
[
    length(Uint8),
    RendezvousInstr1… RendezvousInstrN
]
```
	    </td>
            <td>Rendezvous information. This explains how to contact a Rendezvous Server.</td>
        </tr>
        <tr>
            <td>AppID</td>
            <td>
```json
[
    length(Uint8),	# length in bytes of array
    type(Uint8),	# Type is always ByteArray
    appIdBytes(ByteArray) # Bytearray containing ID
]
```
	    </td>
            <td>AppID values currently in use are allocated by the Intel Client.<sup id="fnref:14"><a class="footnote-ref" href="#fn:14">10</a></sup>
This type value is not currently used, but maintained for backwards compatibility.
</td>
        </tr>
        <tr>
            <td>KeyExchange</td>
            <td>
```json
[
    length(Uint16),
    KexParam(ByteArray) # Parameter A or B (owner is A)
]
```
	    </td>
            <td>Key exchange parameters<sup id="fnref:11"><a class="footnote-ref" href="#fn:11">7</a></sup></td>
        </tr>
        <tr>
            <td>IVData</td>
            <td>
```json
[ length(Uint8), IVData (ByteArray) ]
```
	    </td>
            <td>Cipher Initialization Vector (section <a href="../data-transmission-persistence/index.html#encrypted-message-body">§</a>).  IV encoded as byte array</td>
        </tr>
    </tbody>
</table>

[^7]: Stream Message - Only the Message Body is transmitted for REST protocols.

[^8]: If pkBytes=0, then key is null. Please see section ‎[§](#signatures): Signatures.

[^9]: For more information, please see section ‎[§](#public-key-types): Public Key Types.

[^10]: For more information, please see section ‎[§](#rendezvousinfo): RendezvousInfo.

[^11]: For more information, please see section [§](../protocol-description/#key-exchange-in-the-to2-protocol): Key Exchange in the TO2
Protocol.  Please see section [§](../detailed-protocol-description/#detailed-protocol-description): Data Transmission for more information.

[^12]: For more information, please see section ‎[§](#public-key-encodings): Public Key Encodings.

[^13]: For more information, please see section [§](#rendezvousinfo): RendezvousInfo.

[^14]: For more information, please see section ‎[§](#device-attestation-signature-and-mechanism): EPID 1.1 Signatures.


### Hash Types and HMAC Types

Hash types are based on IANA DNSSEC codes. Please view the following webpage for
more details:

```
http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml
```

HMAC types are the corresponding hash type incremented by 100. The HMAC secret
must be derived by context.


<table>
    <caption>Table 3 - Hash Types and HMAC</caption>
    <thead>
        <tr>
            <th>Hash number</th>
            <th>Hash Algorithm</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>No hash present or hash/HMAC of illegal length
    present.  This value is used only in the Credential Reuse protocol.</td>
        </tr>
        <tr>
            <td>8</td>
            <td>SHA256*</td>
        </tr>
        <tr>
            <td>10</td>
            <td>SHA512</td>
        </tr>
        <tr>
            <td>14</td>
            <td>SHA384</td>
        </tr>
        <tr>
            <td>108</td>
            <td>HMAC-SHA256*</td>
        </tr>
        <tr>
            <td>110</td>
            <td>HMAC-SHA512</td>
        </tr>
        <tr>
            <td>114</td>
            <td>HMAC-SHA384</td>
        </tr>
        <tr>
            <td>Other values</td>
            <td>Reserved</td>
        </tr>
    </tbody>
</table>



The size of the hash and HMAC functions used in the protocol depend on the size
of the keys used for device and owner attestation. Table ‎3‑4 lists the
mapping. The hash and HMAC that are affected by the size of device and
owner attestation keys are listed as follows:

-   Hash of device certificate in Ownership Voucher (OP.hdc)

-   Hash of previous entry in Ownership Voucher entries
    (OP.OwnershipProxyEntry.hp)

-   Hash of header in Ownership Voucher entries (OP.OwnershipProxyEntry.hc)

-   Public key hash in Ownership Credentials (PM.CredOwner.pkh)

-   Hash of to0d object (TO0.OwnerSign.to1d.to0dh)

-   HMAC generated by device (DI.SetHMAC.hmac ,TO2.Done.hmac, and OP.hmac)


<table>
    <caption>Table 4 - Mapping of Hash/HMAC Types with Key Sizes</caption>
    <thead>
        <tr>
            <th>Device Attestation</th>
            <th>Owner Attestation</th>
            <th>Hash and HMAC Types</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Intel<sup>®</sup> EPID</td>
            <td>RSA2048RESTR</td>
            <td>SHA256/HMAC-SHA256</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>RSA2048RESTR</td>
            <td>SHA256/HMAC-SHA256</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>RSA2048RESTR</td>
            <td>SHA384/HMAC-SHA384 (Not a recommended configuration)*</td>
        </tr>
        <tr>
            <td>Intel<sup>®</sup> EPID</td>
            <td>RSA 3072-bit key</td>
            <td>SHA256/HMAC-SHA256</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>RSA 3072-bit key</td>
            <td>SHA256/HMAC-SHA256 (Not a recommended configuration)*</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>RSA 3072-bit key</td>
            <td>SHA384/HMAC-SHA384</td>
        </tr>
        <tr>
            <td>Intel<sup>®</sup> EPID</td>
            <td>ECDSA NIST P-256</td>
            <td>SHA256/HMAC-SHA256</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>ECDSA NIST P-256</td>
            <td>SHA256/HMAC-SHA256</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>ECDSA NIST P-256</td>
            <td>SHA384/HMAC-SHA384 (Not a recommended configuration)*</td>
        </tr>
        <tr>
            <td>Intel<sup>®</sup> EPID</td>
            <td>ECDSA NIST P-384</td>
            <td>SHA384/HMAC-SHA384</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-256</td>
            <td>ECDSA NIST P-384</td>
            <td>SHA384/HMAC-SHA384 (Not a recommended configuration)*</td>
        </tr>
        <tr>
            <td>ECDSA NIST P-384</td>
            <td>ECDSA NIST P-384</td>
            <td>SHA384/HMAC-SHA384</td>
        </tr>
    </tbody>
</table>

---
**Note on not recommended configurations, above**

The Ownership Voucher and the Device key in this configuration have
different cryptographic strengths.  It is recommended that the
strongest cryptographic strength always be used, and that the
strengths match between Device and Owner.

---

### Public Key Types

Public keys types are based on IANA DNSSEC codes, see:

```
http://www.iana.org/assignments/dns-sec-alg-numbers/dns-sec-alg-numbers.xhtml
```

The public key length is derived based on the (explicit) length of the storage
provided for it.


<table>
    <caption>Table 5 - Public Key Number and Associated Algorithm</caption>
    <thead>
        <tr>
            <th>Public Key Number</th>
            <th>Public Key Algorithm</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>No public key present</td>
        </tr>
        <tr>
            <td>1</td>
            <td>RSA2048RESTR (RSA with restrictions imposed by legacy TEE environments)</td>
        </tr>
        <tr>
            <td>2</td>
            <td>DH</td>
        </tr>
        <tr>
            <td>3</td>
            <td>DSA (not currently used)</td>
        </tr>
        <tr>
            <td>4</td>
            <td>RSA_UR (Any valid RSA public key)</td>
        </tr>
        <tr>
            <td>13</td>
            <td>ECDSA P-256</td>
        </tr>
        <tr>
            <td>14</td>
            <td>ECDSA P-384</td>
        </tr>
        <tr>
            <td>90</td>
            <td>Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) v1.0</td>
        </tr>
        <tr>
            <td>91</td>
            <td>Intel<sup>®</sup> EPID v1.1</td>
        </tr>
        <tr>
            <td>92</td>
            <td>Intel<sup>®</sup> EPID v2.0</td>
        </tr>
        <tr>
            <td>113</td>
            <td>ECC P-256 key used for ECDH key exchange (this code is not currently used in the protocol)</td>
        </tr>
        <tr>
            <td>114</td>
            <td>ECC P-384 key used for ECDH key exchange (this code is not currently used in the protocol)</td>
        </tr>
        <tr>
            <td>Other values</td>
            <td>Reserved</td>
        </tr>
    </tbody>
</table>

### Public Key Encodings

The key encoding is listed in the table below.


<table>
    <caption>Table 6 - Encoding Number Table</caption>
    <thead>
        <tr>
            <th>Public Key Encoding</th>
            <th>Encoding Name</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>0</td>
            <td>No public key present</td>
        </tr>
        <tr>
            <td>1</td>
            <td>X.509</td>
        </tr>
        <tr>
            <td>3</td>
            <td>RSAMODEXP – RSA2048RESTR or RSA_UR key with modulus + exponent</td>
        </tr>
        <tr>
            <td>4</td>
            <td>Intel<sup>®</sup> EPID</td>
        </tr>
    </tbody>
</table>

Owner and Server implementations must support all public key encodings, as well
as null keys under X.509 encoding.

#### RSAMODEXP Encoding

In the RSAMODEXP encoding, an RSA key is represented by two byte arrays, one for
the modulus and one for the exponent. The modulus and exponent are based on the
Java\* Big Integer encoding, a big-endian, signed, two’s complement
representation:

-   Big-endian: most significant byte is first, least significant byte is last.

-   Signed encoding: the modulus and exponent are always positive, but the
    encoding can handle negative numbers. This means that the most significant
    bit of the most significant byte (that is, the top bit of byte 0) must match
    the sign. To accomplish this, an extra most significant byte of 0x00 or 0xff
    might need to be pre-pended to the number. For example: 32768 = 0x8000 must
    be represented as three bytes: **0x008000**. *Note that this may require
    removing the first byte transmitted for some crypto packages and key
    values.*

-   Two’s complement: the normal encoding of integers in a computer, where the
    negative of a number is the complement of the bits (aka: one’s complement),
    plus one. For example, 32768 is represented as: **0x008000** (as above).
    Complementing the bits gives **0xff7fff**. Adding one gives: **0xff8000**
    which is -32768. This can be simplified to two bytes: **0x8000**, since the
    sign bit still correctly identifies the number as negative.

1.  *For implementation*, we are aware of crypto packages that use similar
    encodings, except that the modulus and exponent are always positive, so no
    extra sign byte is needed. The above encoding may need to be “tweaked” by
    removing/adding the extra sign byte to work with such packages. For example,
    one package requires that the size of the modulus always be the number of
    bytes which is a power of two (For example, 256 bytes), and the sign byte pushed it
    over by one (to 257 bytes).

### Signatures

A signature is a JSON* object with 3 tags:


-   **"bo"** -- the body to be signed. The signature body is the entire object
    associated with the “bo” tag, exclusive of the tag itself. This object is
    hashed byte-for-byte as transmitted. Note that no spaces or comments are
    permitted in our subset of JSON\*, and that strings are always encapsulated
    with double quotes. In some implementations, it might be necessary to
    reconstruct the original JSON\* from a parsed version to verify the signature.'

-   **"pk"** -- the public key to apply to the signature. Note, when the Public Key
    is null, the “pk” tag must still be present with the value “PKNull” 
    (see section [§](../#composite-types)).

-   **"sg"** -- the signature material itself.

Signatures may be:

-   RSA sign with PKCS1 v1.5 padding, using SHA-256 hash

-   ECDSA NIST P-256.  The signature format uses ASN.1 schema (\[12],
    \[13]) encoded with Distinguished Encoding Rules (DER) \[14].

-   RSA sign with PKCS1 v1.5 padding, using SHA-384 hash (future crypto)

-   ECDSA NIST P-384 (future crypto)

-   Intel<sup>®</sup> EPID 1.0 signatures. The signature format uses ASN.1 schema
    (\[12], \[13]) encoded with Distinguished Encoding Rules (DER)
    \[14]

-   Intel<sup>®</sup> EPID 1.1 signatures

-   Intel<sup>®</sup> EPID 2.0 signatures

Signature generation proceeds as follows:

-   Compute the hash on the body, exactly as you will transmit it.

-   Encrypt the hash using the private key.

-   Include the public key in the signature object, under the “pk” tag.

Signature verification proceeds as follows:

-   Compute the hash on the body, exactly as transmitted.

-   Decrypt the signature with the public key, as provided in the “pk” tag.

-   Verify that the decrypted signature matches the hash.

The body is a single JSON\* item that is terminated by a comma, which in practice
is always a JSON\* object. Thus, the plain text for the signature will be an
object that is *inclusive of* the brace brackets.

**Example:**

```json
{"bo":{"n3":"WpTgdZooMD0="},"pk":[1,3,[257,"AKgIjL … 257 bytes of b64 ...5nCQRk=", 3,"AQAB"]],"sg":[256,"kuL2 ... 256 bytes of b64 … Ps/pfw=="]}
```

The plaintext to be signed is the string: 

```json
{"n3":"WpTgdZooMD0="}
```

### Device Attestation Signature and Mechanism

The Device Attestation signature is used by the Secure Device Onboard Device to prove its
authenticity to the Secure Device Onboard Rendezvous and the Secure Device Onboard Owner. Various
cryptographic signing mechanisms can be used for this purpose. In this
specification, we present mechanisms based on Intel<sup>®</sup> Enhanced Privacy Identifier
(Intel<sup>®</sup> EPID) signatures and Elliptical Curve Digital Signature Algorithm (ECDSA)
signatures.

In some cases, a signature cannot stand alone, but requires some protocol
interaction to prepare for it. For example, a multi-application Trusted
Execution Environment (TEE) may need to negotiate program instance parameters
with the verifier before the signature can be trusted. In particular, Intel<sup>®</sup> EPID
signers need to obtain revocation information and cooperatively sign “proofs”.
For this reason, the Device Attestation mechanism exchanges the following three
messages:

- **eA** -- from device to signature verifier, provides initial device based information

- **eB** -- from verifier to device, provides a response to eA

- **sg** -- from device to verifier, contains the actual signature

The eA and eB messages are structured as “SigInfo”, and contain a field that
identifies the signature type as defined in Table ‎3‑7.


<table>
    <caption>Table 7 - Signatures Types</caption>
    <thead>
        <tr>
            <th>Encoding</th>
            <th>Type Name</th>
            <th>Type Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>13</td>
            <td>ECDSA P-256</td>
            <td>ECDSA P-256 signature with SHA-256 hash</td>
        </tr>
        <tr>
            <td>14</td>
            <td>ECDSA P-384</td>
            <td>ECDSA P-384 signature with SHA-384 hash</td>
        </tr>
        <tr>
            <td>90</td>
            <td>EPID10</td>
            <td>Intel<sup>®</sup> EPID 1.0 signature information for the Intel Client environment. Sufficient information to generate the Intel Client “EPID provisioning” call.</td>
        </tr>
        <tr>
            <td>91</td>
            <td>EPID11</td>
            <td>Intel<sup>®</sup> EPID 1.1 signature information for the Intel Client environment. Sufficient information to generate the Client “EPID provisioning” call.</td>
        </tr>
        <tr>
            <td>92</td>
            <td>EPID20</td>
            <td>Intel<sup>®</sup> EPID 2.0 signature information for non-Intel Client implementation.</td>
        </tr>
    </tbody>
</table>

#### Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) Signatures Overview

Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) signatures are generated by Devices
that support a Hardware Root of Trust based on Intel<sup>®</sup> EPID 1.0, Intel<sup>®</sup> EPID 1.1,
or Intel<sup>®</sup> EPID 2.0.

1.  Intel<sup>®</sup> EPID 2.0 is implemented at Intel with both 32-bit and 128-bit group
    IDs. This specification assumes 128-bit group IDs for Intel<sup>®</sup> EPID. Intel<sup>®</sup> EPID
    32-bit group IDs are supported by extending the 32-bit group ID to 128 bits
    by zero padding to the left (most significant bits), thus mapping the 32-bit
    group space into the 128-bit group space. In some cases, the Secure
    Device Onboard implementation must convert group IDs back and forth for
    platform software that requires 32-bit group IDs. To our knowledge,
    platforms where the Intel<sup>®</sup> EPID 2.0 software supports only 32-bit group IDs
    are matched by hardware provisioned with Intel<sup>®</sup> EPID keys with 32-bits.

The EPID signer needs Intel<sup>®</sup> EPID revocation information in order to generate a
valid Intel<sup>®</sup> EPID signature from an Intel<sup>®</sup> EPID private key. In particular, the
signer needs the signature revocation list (SIGRL) for its group. The SIGRL may
be obtained from Intel’s iKGF web sites, but it is easiest to obtain using the
Intel<sup>®</sup> EPID Verification Service.

In some platforms, Intel<sup>®</sup> EPID revocation information is downloaded using other
tools. However, for consistency, Secure Device Onboard allows the Device to download this
information as part of the TO1 and TO2 Protocols.

Intel<sup>®</sup> EPID information is encoded in the SigInfo structures eA, eB, and in the
pkType and sg fields of the signature itself. Typically eA and eB are in
subsequent messages, with the Intel<sup>®</sup> EPID signed message coming just afterwards.

#### Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) 1.1 Signatures (type EPID11)

Intel<sup>®</sup> EPID 1.1 signature information is encoded using the EPID11 signature type,
using the “eA”, “eB” and “sg” fields, each from separate messages. The contents
of each field is as follows:

-   eA encodes the EPID group ID as 4 bytes, in network byte order (MSB first):


<table>
    <caption>Table 8 - eA Encoding, type EPID11</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[4]</td>
            <td>groupId</td>
            <td>EPID 1.1 group ID</td>
        </tr>
    </tbody>
</table>

-   eB encodes the EPID certificate and other items needed by the Intel Client. The SIGRL
    is inside the group certificate. Length values (UInt16) are encoded in
    network order (MSB first):


<table>
    <caption>Table 9 - eB Encoding, type EPID11</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>UInt16</td>
            <td>groupCertSigma10Size</td>
            <td>Size of data in groupCertSigma10 field</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>groupCertSigma10</td>
            <td>Legacy group certificate (binary format)</td>
        </tr>
        <tr>
            <td>UInt16</td>
            <td>groupCertSigma11Size</td>
            <td>Size of data in groupCertSigma11 field</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>groupCertSigma11</td>
            <td>X.509 group certificate</td>
        </tr>
        <tr>
            <td>UInt16</td>
            <td>sigRLSize</td>
            <td>Size of data in sigRL field (in bytes)</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>sigRL</td>
            <td>SigRL as given below</td>
        </tr>
    </tbody>
</table>

-   If there is no SIGRL, sigRLSize is zero, and sigRL is empty (not present).

!!! Note 
    sigRL non-zero implies that n2 &gt; 0 (below).

-   The sigRL format is as follows (all fields are encoded in network order (MSB
    first)):


<table>
    <caption>Table 10 - SigRL Format</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Field Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>UInt16</td>
            <td>sver</td>
            <td>Intel<sup>®</sup> EPID version number. Must be 0x0001 for Intel<sup>®</sup> EPID1.1</td>
        </tr>
        <tr>
            <td>UInt16</td>
            <td>blobID</td>
            <td>ID of the data type. Must be 0x000e for SigRL</td>
        </tr>
        <tr>
            <td>UInt32</td>
            <td>gid</td>
            <td>Group ID</td>
        </tr>
        <tr>
            <td>UInt32</td>
            <td>RLver</td>
            <td>Revocation list version number</td>
        </tr>
        <tr>
            <td>UInt32</td>
            <td>n2</td>
            <td>Number of entries in SigRL</td>
        </tr>
        <tr>
            <td>BYTE[64] (n2 of them)</td>
            <td>B[i] for i in range [0; n2-1]</td>
            <td>Bi elements of G3</td>
        </tr>
        <tr>
            <td>BYTE[64] (n2 of them)</td>
            <td>K[i] for i in range [0; n2-1]</td>
            <td>Ki elements of G3</td>
        </tr>
        <tr>
            <td>BYTE[64]</td>
            <td>sig</td>
            <td>512-bit ECDSA signature on the revocation list signed by the issuer using Intel Signing Key (ISK). **Contact Secure Device Onboard Enablement team for details.** </td>
        </tr>
    </tbody>
</table>

-   sg encodes the signature, according to the Intel<sup>®</sup> EPID1.1 specification. The
    length is given in the message format:


<table>
    <caption>Table 11 - sg Encoding, type EPID11</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[]</td>
            <td>signature</td>
            <td>Intel<sup>®</sup> EPID 1.1 signature according to the Intel<sup>®</sup> EPID 1.1 specification</td>
        </tr>
    </tbody>
</table>

The data being signed is:


<table>
    <caption>Table 12 - Data Signatures, type EPID11</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[48]</td>
            <td>Prefix</td>
            <td>All zeros, except: Prefix[4]=0x48 and Prefix[8]=0x8</td>
        </tr>
        <tr>
            <td>BYTE[16]</td>
            <td>ID</td>
            <td>App-ID, same as in message body (“ai” tag in messages: TO1.ProveToSDO and TO2.ProveDevice)</td>
        </tr>
        <tr>
            <td>BYTE[16]</td>
            <td>Zero-padding</td>
            <td>Zeros</td>
        </tr>
        <tr>
            <td>BYTE[16]</td>
            <td>Nonce</td>
            <td>Nonce value same as in message body (“n4” tag in messages: TO1.ProveToSDO and “n6” tag in TO2.ProveDevice)</td>
        </tr>
        <tr>
            <td>BYTE[16]</td>
            <td>Zero-padding</td>
            <td>Zeros (used to allow Nonce to be 32 bytes)</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>Message body</td>
            <td>As in this protocol specification, from open to close brace bracket of JSON* text.</td>
        </tr>
    </tbody>
</table>

1.  Rendezvous Service and Owner must (logically) prefix the contents of the
    message with specific data (as above), before doing Intel<sup>®</sup> EPID verification.

#### Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) 2.0 Signatures (non-Intel Client, type EPID20)

Intel<sup>®</sup> EPID 2.0 signatures are encoded using “eA”, “eB” and the “sg” fields, each
from separate messages. The contents of each field is as follows:

-   eA encodes the EPID group ID as 16 bytes, in network byte order (MSB first).
    Hosts which use Intel<sup>®</sup> EPID2.0 with 32-bit group IDs must zero pad the top 96
    bits.


<table>
    <caption>Table 13 - eA Encoding, type EPID20</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[16]</td>
            <td>groupId</td>
            <td>Intel<sup>®</sup> EPID 2.0 group ID</td>
        </tr>
    </tbody>
</table>

-   eB encodes the signature revocation list (sigRL) and EPID group public key.
    Length values (UInt16) are encoded in network order (MSB first):


<table>
    <caption>Table 14 - eB Encoding, type EPID20</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>UInt16</td>
            <td>sigRLSize</td>
            <td>Size of data in sigRL field</td>
        </tr>
        <tr>
            <td>BYTE[sigRLSize]</td>
            <td>sigRL</td>
            <td>SigRL according to Intel<sup>®</sup> EPID 2.0 specification</td>
        </tr>
        <tr>
            <td>UInt16</td>
            <td>publicKeySize</td>
            <td>Size of data in publicKey field</td>
        </tr>
        <tr>
            <td>BYTE[publicKeySize]</td>
            <td>publicKey</td>
            <td>Group public key according to Intel<sup>®</sup> EPID 2.0 specification</td>
        </tr>
    </tbody>
</table>

-   sg encodes the signature, according to the Intel<sup>®</sup> EPID 2.0 specification. The
    length is given in the message format.


<table>
    <caption>Table 15 - sg Encoding, type EPID20</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[]</td>
            <td>signature</td>
            <td>Intel<sup>®</sup> EPID 2.0 signature according to Intel<sup>®</sup> EPID 2.0 specification</td>
        </tr>
    </tbody>
</table>

#### Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) 1.0 Signatures (type EPID10)

Intel<sup>®</sup> EPID 1.0 signature information is encoded using the EPID10 signature type,
using the “eA”, “eB” and “sg” fields, each from separate messages. The contents
of each field is as follows:

-   eA encodes the EPID group ID as 4 bytes, in network byte order (MSB first):


<table>
    <caption>Table 16 - eA Encoding, type EPID10</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[4]</td>
            <td>groupId</td>
            <td>Intel<sup>®</sup> EPID 1.0 group ID</td>
        </tr>
    </tbody>
</table>

-   eB encodes the Intel<sup>®</sup> EPID certificate and other items needed by the Intel Client. The
    SIGRL is inside the group certificate. Length values (UInt16) are encoded in
    network order (MSB first):


<table>
    <caption>Table 17 - eB Encoding, type EPID10</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>UInt16</td>
            <td>groupCertSigma10Size</td>
            <td>Size of data in groupCertSigma10 field</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>groupCertSigma10</td>
            <td>Legacy group certificate (binary format)</td>
        </tr>
    </tbody>
</table>

-   sg encodes the signature, according to the Intel<sup>®</sup> EPID 1.0 specification. The
    length is given in the message format:


<table>
    <caption>Table 18 - sg Encoding, type EPID10</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE[]</td>
            <td>Signature</td>
            <td>Intel<sup>®</sup> EPID 1.0 signature according to Intel<sup>®</sup> EPID 1.0 specification</td>
        </tr>
    </tbody>
</table>

The data being signed is:


<table>
    <caption>Table 19 - Data Signatures, type EPID10</caption>
    <thead>
        <tr>
            <th>Type</th>
            <th>Name</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BYTE</td>
            <td>ID-length</td>
            <td>Length of ID field</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>ID</td>
            <td>App-ID, same as in message body (“ai” tag in messages: TO1.ProveToSDO and TO2.ProveDevice)</td>
        </tr>
        <tr>
            <td>BYTE[16]</td>
            <td>Nonce</td>
            <td>Nonce value same as in message body (“n4” tag in messages: TO1.ProveToSDO and “n6” tag in TO2.ProveDevice)</td>
        </tr>
        <tr>
            <td>BYTE[]</td>
            <td>Message body</td>
            <td>As in this protocol specification, from open to close brace bracket of JSON text.</td>
        </tr>
    </tbody>
</table>

1.  Rendezvous Service and Owner must (logically) prefix the contents of the
    message with specific data (as above), before doing Intel<sup>®</sup> EPID verification.

#### ECDSA NIST P-256 and ECDSA NIST P-384 Signatures

The ECDSA attestation is used for devices where the Intel<sup>®</sup> EPID privacy
mechanisms are not needed. The Device contains an ECDSA private key, and the
Ownership Voucher is enhanced to give the Owner access to the public key. The
public key is typically encoded in an X.509 certificate, signed by one or more
Certificate Authorities. The private key is used to sign the device attestation.
The public key is used to verify the signature.

For ECDSA attestation, the Length field in eA and eB encoding is set to 0 and
Info field is not present (zero length). ECDSA NIST P-256 uses SHA-256 hash, and
ECDSA NIST P-384 uses SHA-384 hash.

For ECDSA, both the private key and the public key are Device identities. This
means that the Secure Device Onboard Rendezvous Service and Secure Device Onboard Owner who verify the
Device attestation signature receive a unique and permanent identity for the
device, even before they verify the signature. This information can be used to
trace the subsequent owners of the device. For this reason, we recommend careful
administrative measures to ensure that this information is used securely and
discarded appropriately.

### Null Public Keys

A public key in the “pk” tag may be left as null. This is done when the public
key is already known to the verifying party, because it is a requirement that it
know it, or because it was already transmitted earlier.

Intel<sup>®</sup> EPID public keys encode only the group number in the “pk” tag, since this
is sufficient to identify the public key to the Intel back end.

Null public keys are used for some testing stubs.

### RendezvousInfo

The RendezvousInfo type indicates the manner and order in which the Device and
Owner find the Rendezvous Server. It is configured during manufacturing (For example,
at an ODM), so the manufacturing entity has the choice of which Rendezvous
Server to use and how to access it.

RendezvousInfo consists of a sequence of rendezvous instructions which are
interpreted in order during the TO0 and TO1 Protocols (JSON* sequence of object,
such as [{t1:v1,…},…,{tn,…}]). The tags are encoded in alphabetical/character
encoding order.

Unlike other Secure Device Onboard JSON* objects, JSON\* tags appear only for instructions that
differ from the default. Other variables assume default values. A default value
of “none” indicates that the variable is assumed not present in the instruction
unless its tag appears explicitly.

Each set of rendezvous instructions (that is, each element of the JSON\* sequence) is
interpreted as one set of instructions for reaching the Rendezvous Server, with
differing conditions. For example, one set might indicate using the wireless
interface, and another set might indicate using the wired interface. It is
possible that a given set corresponds to multiple ways to access the Rendezvous
Server (For example, multiple IP addresses that correspond to a single DNS name), and
these must all be tried before the Device or Owner moves to the next element of
the sequence.

The Owner and Device process the RendezvousInfo, attempting to access the
Rendezvous Server. The first successful connection may be used.

The Device and Owner may avoid obviously redundant operations, such as
contacting the same IP address twice when a DNS name maps to an IP address
explicit in a separate rendezvous instruction.

To execute each rendezvous instruction, the program defines a set of variables,
one for each JSON* tag in the RendezvousInfo instructions. It initializes all
variables to default values. Then the rendezvous instruction is interpreted, and
updates each variable. If the user input variable is set to true, and user input
is available, the user is allowed to update each variable. On constrained
devices, some variables do not exist. The constrained implementation interprets
each instruction as if this variable was not present.

Some variables apply only to the Owner and some only to the Device. See the
table below. When a variable does not apply, it is interpreted as if it had
never been specified. This means that the Owner can only ever notice the tags:
“only”, “ip”, “pow”, “delaysec”, and “dn”. This is because the Owner, as a
cloud-based server, is expected to use normal Internet rules to access the
Rendezvous Service. The Device, which may be in a specialized network and may be
constrained, might need additional parameters.

The “only” (Only for) and "delaysec" tags have side effects.

-   When "only":"dev" appears in a set of instructions, an Owner must skip the
    entire set

-   When "only":"owner" appears in a set of instructions, a Device must skip the
    entire set

-   When "delay":UInt32 appears in a set of instructions, the set is followed by
    a delay for the number of seconds specified, increased or decreased by a
    random value up to 25%. An instruction that contains just a “delay”, can be
    appended to the end of the RendezvousInfo to force a particular randomized
    delay before retrying the entire sequence.


<table>
    <caption>Table 20 - RendezvousInfo Variables</caption>
    <thead>
        <tr>
            <th>Rendezvous Instruction Variable</th>
            <th>Default Value</th>
            <th>Variable Encoding Tag</th>
            <th>Variable Type</th>
            <th>Required</th>
            <th>Device</th>
            <th>Owner</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Only for</td>
            <td>None</td>
            <td>"only"</td>
            <td>"dev" or "owner"</td>
            <td>No</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>IP address</td>
            <td>None</td>
            <td>"ip"</td>
            <td>IPAddress</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>Port, Device</td>
            <td>Based on protocol</td>
            <td>"po"</td>
            <td>UInt16</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Port, Owner</td>
            <td>Based on protocol</td>
            <td>“pow”</td>
            <td>UInt16</td>
            <td>No</td>
            <td>No</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>DNS name</td>
            <td>None</td>
            <td>"dn"</td>
            <td>String</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
        <tr>
            <td>TLS Server cert hash</td>
            <td>None</td>
            <td>"sch"</td>
            <td>Hash</td>
            <td>No</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>TLS CA cert hash</td>
            <td>None</td>
            <td>"cch"</td>
            <td>Hash</td>
            <td>No</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>User input</td>
            <td>No</td>
            <td>"ui"</td>
            <td>UInt8 (0/1)</td>
            <td>No</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>SSID</td>
            <td>None</td>
            <td>"ss"</td>
            <td>String</td>
            <td>If has Wi-Fi*</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Wireless Password</td>
            <td>None</td>
            <td>"pw"</td>
            <td>String</td>
            <td>If has Wi-Fi*</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Wireless Security Password</td>
            <td>Chosen Automatically</td>
            <td>"wsp"</td>
            <td>String</td>
            <td>If has Wi-Fi*</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Medium</td>
            <td>*Device dependent*</td>
            <td>"me"</td>
            <td>String</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Protocol</td>
            <td>TLS</td>
            <td>"pr"</td>
            <td>String</td>
            <td>No</td>
            <td>Yes</td>
            <td>No</td>
        </tr>
        <tr>
            <td>Delay</td>
            <td>0</td>
            <td>“delaysec”</td>
            <td>UInt32</td>
            <td>Yes</td>
            <td>Yes</td>
            <td>Yes</td>
        </tr>
    </tbody>
</table>

Rendezvous Instruction (RendezvousInstr) entries are specified as having the
variables in alphabetical order. This does not affect the interpretation of
these variables, but makes the computation of a signature that includes them
simpler for some implementations.

-   If the “only” element appears, and the value does not match the interpreting
    entity (Device/Owner), this instruction is terminated and control proceeds
    with the next set of instructions.

-   Medium is selected (“me”). The Device uses the selected preferred medium, or
    terminates this instruction. When no medium is specified, the Device may
    establish its own preference, perhaps based on the other parameters (for
    example, TLS might be available on one medium, but not another).

-   The protocol is chosen (“pr”), if it is supported. Otherwise, the
    instruction terminates. The Owner always chooses the protocol “https”.

-   The Device attempts to resolve the DNS address. If DNS query is successful,
    then the resolved IP addresses are tried one after another.

-   Else, if DNS resolution fails or the Device fails to communicate with all of
    the resolved IP addresses, then the specified IP address is used as the
    target IP address.

-   Else, the instruction terminates.

-   If TLS is used on a Device (pr=https or pr=tls), the hash instructions are
    used as specified against server certificates appearing in the TLS handshake
    (or, for ps=https, the underlying TLS connection’s handshake). This applies
    to the Device only.

-   If the server certificate hash is specified (on a Device), the server’s
    certificate is extracted from the certificate chain, SHA256 hash computed,
    and compared to the specified value. Failure to match causes the TLS
    authentication to fail.

-   If the CA certificate hash is specified (on a Device), the other
    certificates in the server certificate chain are extracted one by one,
    SHA256 hash computed, and compared to the specified value. Any match allows
    TLS authentication to proceed. No match causes TLS authentication to fail.

-   The Owner always applies usual CA trust to server certificates used in TLS
    for the TO0 Protocol.

-   Attempt as many connections as are implied by the set of variables
    established and choices made, above. For example, try each of the addresses
    returned by a DNS query if there is a valid DNS name.

-   If a “delaysec” tag appears, delay as specified.

-   If “delaysec” does not appear and the last entry in RendezvousInfo has been
    processed, a delay of 120s ± random(30) is executed.

Medium values:

<table>
<tbody>
    <tr>
        <td>"eth0".."eth9"
	</td>
        <td>
mapped to first through 10th wired Ethernet interfaces.
These interfaces may appear with different names in a given platform.
	</td>
    </tr>
    <tr>
        <td>"eth\*"
	</td>
        <td>
means to try as many wired interfaces as makes sense for this
platform, in any order. For example, a device which has one or more wired
interfaces that are configured to access the Internet (For example, “wan0”) might
use this configuration to try any of them that has Ethernet link.
	</td>
    </tr>
    <tr>
        <td>"wifi0" .. "wifi9"
	</td>
        <td>
mapped to first through 10th Wi-Fi* interfaces. These
interfaces may appear with different names in a given platform.
	</td>
    </tr>
    <tr>
        <td>"wifi\*"
	</td>
        <td>
means to try as many Wi-Fi* interfaces as makes sense for this
platform, in any order
	</td>
    </tr>
    <tr>
        <td>*Others*
	</td>
        <td>*device dependent.*
	</td>
    </tr>

</tbody>
</table>

Protocol Values:

<table>
  <tbody>
    <tr>
        <td>"rest":
	</td>
        <td>
first supported protocol from:
```
HTTPS
HTTP
CoAP/TCP
```
	</td>
    </tr>
    <tr>
        <td>"tcp":
	</td>
        <td>bare TCP, if supported
	</td>
    </tr>
    <tr>
        <td>"tls":
	</td>
        <td>bare TLS, if supported
	</td>
    </tr>
    <tr>
        <td>"CoAP/tcp":
	</td>
        <td>CoAP protocol over tcp, if supported
	</td>
    </tr>
    <tr>
        <td>"http":
	</td>
        <td>HTTP over TCP
	</td>
    </tr>
    <tr>
        <td>"https":
	</td>
        <td>HTTP over TLS, if supported
	</td>
    </tr>
  </tbody>
</table>

#### Example of RendezvousInfo: Different Ports for Device and Owner

```json
…"r":[1,{"dn":"mpservice.net","po":80,"pow":443}]…
```

On both Device (TO1 Protocol) and Owner (TO0 Protocol), attempt to connect to
all IP addresses returned by the DNS query for “mpservice.net”. The Owner
queries TO0 Protocol on port 443 (Owner always uses TLS). The Device queries TO1
Protocol on port 80 (might be HTTP or HTTPS, or both, depending on the device
default).

In the following situation, the Device needs to try Wi-Fi\* media, as many as it
can connect to. The Owner just uses the DNS name without the Wi-Fi\* media
specification, because “me” applies only to Devices, and is thus ignored by the
Owner.

```json
[1,{"me":"wifi\*","dn":"mpservice.net"}]
```

The above is thus equivalent to:

```json
[2,[3, {"only":"dev","me":"wifi\*","dn":"mpservice.net"}],
   [2{"only":"owner","dn":"mpservice.net"}]]
```

(Inserted newline for formatting purposes)

In the above, “eth*” could be used for wired interfaces that are purposed for
Internet access, such as the *outside* wired interface on a gateway or router.

### ServiceInfo and Management Service – Agent Interactions

The ServiceInfo type is a collection of key-value pairs which allows an
interaction between the Management Service (on the cloud side) and Management
Agent functions (on the device side), using the Secure Device Onboard encrypted channel as a
transport.

Conceptually, each key-value pair is a message between a module in the Owner and
a module on the Device that implements some primitive function. Messages have a
name and a value. The ServiceInfo key is the module name and the message name,
separated by a colon. ServiceInfo is constrained to a subset of printable ASCII,
but base64 or other encoding may be used to encode any textual or binary
formats.

<table>
    <thead>
        <tr>
            <th>ServiceInfo Key</th>
            <th>ServiceInfo Value</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <td>
moduleName:messageName where both moduleName and messageName are
printable ASCII characters containing no Unicode escapes (&bsol;uXXXX) or
the characters: {}[]&amp;&quot;&bsol; (see Table 1 in section <a href="../protocol-description/#json-distinguished-encoding">§</a>). Where appropriate,
the moduleName may contain a version of the module. By convention, the
version is separated using a hyphen character (‘-‘), as in: tpm-1.2 or
tpm-2.

          </td> 
          <td>

Printable ASCII characters containing no Unicode escapes (&bsol;uXXXX) or
the characters: {}[]&amp;&quot;&bsol; (see Table 1 in section <a href="../protocol-description/#json-distinguished-encoding">§</a>) Individual or
lists of Numbers and selectors (enum type strings) may be expressed
directly in ASCII. Textual and binary data should be encoded in
base64. Other encodings are permissible on a module-by-module basis,
with the above limits.
	  </td> 

        </tr>
    </tbody>
</table>

Messages sent to a module on the Secure Device Onboard Device may interact with the Device
OS to install software components. Another message might use those components in
combination with a cryptographic key, to establish communications. In some
systems, the Management Agent might be installed by cooperating modules before
it is active by others, allowing an “off-the-shelf” device to be customized by
Secure Device Onboard.

The intention is that modules will implement common or standardized IOT
provisioning functions, and will be reused for different IOT solutions
provisioned by Secure Device Onboard.

In some cases, modules on the Secure Device Onboard Owner and Secure Device Onboard Device will be
designed to cooperate directly with each other. For example, a module that
implements a particular device management client on the Secure Device Onboard Device, and
its counterpart that feeds it exactly the right credentials on the Secure Device Onboard
Owner. In other cases, modules may implement IOT or OS primitives so that the
Secure Device Onboard Owner or Secure Device Onboard Device picks and chooses among them. For example,
allocating a key pair on the Secure Device Onboard Device; signing a certificate on the
Secure Device Onboard Owner; transferring a file into the OS; upgrading software; and so on.

The following set of examples is presented for illustrative purposes, and is not
intended to constrain Secure Device Onboard implementations in any way:

<table>
    <thead>
        <tr>
            <th>Example Key</th>
            <th>Example Value</th>
            <th>Comment</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>firmware_update:active</td>
            <td>1</td>
            <td>Hypothetical firmware update module, sends “active” message with value of 1 (true)</td>
        </tr>
        <tr>
            <td>firmware_update:codeSize</td>
            <td>262144</td>
            <td>Code size is 256k</td>
        </tr>
        <tr>
            <td>firmware_update:code001</td>
            <td>Base64 data</td>
            <td>First 512 bytes of firmware update, encoded in base64</td>
        </tr>
        <tr>
            <td>firmware_update:verify</td>
            <td>Base64 data</td>
            <td>SHA-384 of firmware image</td>
        </tr>
        <tr>
            <td>wget:hashSha384</td>
            <td>Base64 data</td>
            <td>SHA-384 hash of data that is downloaded in the next message</td>
        </tr>
        <tr>
            <td>wget:CAfiles.dat</td>
            <td>http://myserver/CAfiles.dat</td>
            <td>Download CA database</td>
        </tr>
        <tr>
            <td>cmd-linux:\#!/bin/sh</td>
            <td>exec /usr/local/bin/mydaemon -k mykeyfile.pkcs7 -ca CAfiles.dat</td>
            <td>Hypothetical module to execute Linux* scripting commands, message name gives interpreter, value gives shell code</td>
        </tr>
    </tbody>
</table>

The API between the Management Agent / Device OS and the Secure Device Onboard Device, and
between the Management Service and the Secure Device Onboard Owner, are outside the scope of
this document. The requirements for this API are as follows:

-   A mechanism to discover modules on the Secure Device Onboard Owner and to establish a
    preference among them (analogous to a preference for TPM2 over TPM1.2)

-   A mechanism to connect modules on the Secure Device Onboard Device. A complex Secure Device Onboard
    Device might be able to discover modules, but a simpler device could have
    modules “hard” coded

-   A mechanism to generate Secure Device Onboard messages to modules. As above, modules can
    send messages to their counterparts or to other modules

-   On constrained Secure Device Onboard Devices, common code for performing base64 decoding
    is desirable.

    Common code for modules to store and buffer state from messages is
    desirable.

    On complex Secure Device Onboard Devices, the ability of modules to send messages to
    each other may also be supported. For example, a file transfer module and a
    file storage module might be called as primitives for a “file transfer and
    store” module.

    One special case of the above. We envision a “TEE” module, that encapsulates
    messages for other modules, but causes an error unless these modules are
    implemented at the same security level as the Secure Device Onboard implementation
    (For example, in the same Trusted Execution Environment). For example,
    TEE:tpm:createkey causes an error if the module called “tpm” is not at the
    same security level as Secure Device Onboard. This module might encrypt data to ensure
    that it can only be processed in a trusted environment. Implementations
    which support multiple security levels for code execution should allow for
    this function, since this capability cannot be simulated using other Secure Device Onboard mechanisms.

<figure id="fig-AgentInteraction">
    <!--    <img src="../img/d27add553925cda2902241dcbe4e9349.emf" alt="Agent Interaction"/> -->
    <img src="../img/agent-interaction.png" alt="Agent Interaction"/>
    <figcaption>Management Service - Agent Interactions via ServiceInfo</figcaption>
</figure>


#### Mapping Messages to ServiceInfo

A module may have an arbitrary number of messages. There are no arrays, but
numbered message names can be used to simulate their effect (mod:key1,
mod:key2).

Since ServiceInfo messages are separated into one or more Secure Device Onboard messages, it
is possible to use the same message over and over again. Whether this is has a
cumulative or repetitive effect is up to the module that interprets the messages
(For example, file:part might be repeated for successive parts, but tpm:certificate
might be individual certificates).

Messages are processed in the order they appear in ServiceInfo.

The same message may also be repeated in a single ServiceInfo.

ServiceInfo does not have to be interpreted as it is parsed from the message. It
is legal to buffer the entire ServiceInfo and interpret it all later. However,
the messages must be interpreted in the same order.

#### The SDO Device Module

The “sdodev” module implements a set of messages to the Secure Device Onboard Owner that
identify the capabilities of the device. All Secure Device Onboard Owners must implement
this module, and Secure Device Onboard Owner implementations must provide these messages to
any module asks for them. The “sdodev” messages must appear first in the Device
ServiceInfo.

The following messages are defined in the SDO Device Module:


<table>
    <caption>Table 21 - sdodev Module Device Service Info Keys</caption>
    <thead>
        <tr>
            <th>Device Service Info Key</th>
            <th>Disposition</th>
            <th>Meaning / Action</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>sdodev:os</td>
            <td>Required</td>
            <td>OS name (For example, Linux*)</td>
        </tr>
        <tr>
            <td>sdodev:arch</td>
            <td>Required</td>
            <td>Architecture name / instruction set (For example, X86_64)</td>
        </tr>
        <tr>
            <td>sdodev:version</td>
            <td>Required</td>
            <td>Version of OS (For example, “Ubuntu* 16.0.4LTS”)</td>
        </tr>
        <tr>
            <td>sdodev:device</td>
            <td>Required</td>
            <td>Model specifier for this Secure Device Onboard Device, manufacturer specific</td>
        </tr>
        <tr>
            <td>sdodev:sn</td>
            <td>Optional</td>
            <td>Serial number for this Secure Device Onboard Device, manufacturer specific</td>
        </tr>
        <tr>
            <td>sdodev:pathsep</td>
            <td>Optional</td>
            <td>Filename path separator, between the directory and sub-directory (For example, ‘/’ or ‘&bsol;’)</td>
        </tr>
        <tr>
            <td>sdodev:sep</td>
            <td>Required</td>
            <td>Filename separator, that works to make lists of file names (For example, ‘:’ or ‘;’)</td>
        </tr>
        <tr>
            <td>sdodev:nl</td>
            <td>Optional</td>
            <td>Newline sequence (For example, “&bsol;u000a” or “&bsol;u000d&bsol;u000a”)</td>
        </tr>
        <tr>
            <td>sdodev:tmp</td>
            <td>Optional</td>
            <td>Location of temporary directory, including terminating file separator (For example, “/tmp”)</td>
        </tr>
        <tr>
            <td>sdodev:dir</td>
            <td>Optional</td>
            <td>Location of suggested installation directory, including terminating file separator (For example, “.” or “/home/sdo” or “c:&bsol;Program Files&bsol;SDO”)</td>
        </tr>
        <tr>
            <td>sdodev:progenv</td>
            <td>Optional</td>
            <td>Programming environment. See Table ‎22</td>
        </tr>
        <tr>
            <td></td>
            <td></td>
            <td>(For example, “bin:java:py3:py2”)</td>
        </tr>
        <tr>
            <td>sdodev:bin</td>
            <td>Required</td>
            <td>Either the same value as “arch”, or a list of machine formats that can be interpreted by this device, in preference order, separated by the “sep” value</td>
        </tr>
        <tr>
            <td></td>
            <td></td>
            <td>(For example, “x86:X86_64”)</td>
        </tr>
        <tr>
            <td>sdodev:mudurl</td>
            <td>Optional</td>
            <td>URL for the Manufacturer Usage Description file that relates to this device (See CR023)</td>
        </tr>
        <tr>
            <td>sdodev:modules</td>
            <td>Optional</td>
            <td>A list of module names supported by this Secure Device Onboard Device, separated by the separator specified in “sdodev:sep”</td>
        </tr>
    </tbody>
</table>

The “progenv” key-value is used to indicate the Device’ capabilities for running
programs. This is a list of tags, separated by the “sep” value, that indicates
which programming environments are available and preferred on this platform.
For example, bin;perl;cmd means use system binary format (preferred), but Perl\* is
also supported, and Windows\* CMD shell is also supported, but Perl\* is preferred
over CMD.

The following tags are supported at present. Version numbers may be appended to
the tag (as in py2 and py3).


<table>
    <caption>Table 22 - sdodev Module "progenv" Key Tags</caption>
    <thead>
        <tr>
            <th>“progenv” tag</th>
            <th>Meaning</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>bin</td>
            <td>System-dependent most common binary format (the “arch” key-value may inform the format)</td>
        </tr>
        <tr>
            <td>java</td>
            <td>Java* class/jar (openJDK* compatible)</td>
        </tr>
        <tr>
            <td>js</td>
            <td>Node.js* (Javascript*)</td>
        </tr>
        <tr>
            <td>py2</td>
            <td>Python* version 2</td>
        </tr>
        <tr>
            <td>py3</td>
            <td>Python* version 3</td>
        </tr>
        <tr>
            <td>perl*</td>
            <td>Perl* 5</td>
        </tr>
        <tr>
            <td>bash</td>
            <td>Bourne shell</td>
        </tr>
        <tr>
            <td>ksh</td>
            <td>KornShell*</td>
        </tr>
        <tr>
            <td>sh</td>
            <td>*NIX system shell (whichever one it is)</td>
        </tr>
        <tr>
            <td>cmd</td>
            <td>Windows* CMD</td>
        </tr>
        <tr>
            <td>psh</td>
            <td>Windows PowerShell*</td>
        </tr>
        <tr>
            <td>vbs</td>
            <td>Windows Visual Basic* Script</td>
        </tr>
        <tr>
            <td>…</td>
            <td>(Other specifiers may be defined on request. Contact the Secure Device Onboard Enablement team)</td>
        </tr>
    </tbody>
</table>

#### Module Selection

In some Secure Device Onboard implementations, multiple modules can perform overlapping
functions. Some modules may implement legacy versions of others (For example, TPM
versions) and some modules may implement alternative IOT control techniques
(For example, MQTT versus CoAP). The Secure Device Onboard Owner and Secure Device Onboard Device need to
negotiate to select the right set of modules.

In this version of Secure Device Onboard, module selection is as follows:

1.  The
    [TO2.GetNextDeviceServiceInfo.psi]  
    variable lists modules supported by the Secure Device Onboard Owner in order of Owner
    preference.

2.  The [Device ServiceInfo] indicates which
    modules are selected by including an “active” message and provides
    device-side data for the modules from the Management Agent.

3.  The [Owner ServiceInfo] may deselect modules
    with its own “active” messages, and provides messages containing owner-side
    data from the Management Service.

If possible, all module functions should complete in time to allow the Secure Device Onboard
operation to succeed or fail based on module operation, so that a module failure
causes the entire Secure Device Onboard operation to fail and be retried later. In some
cases, the module cannot determine success criteria before Secure Device Onboard completes
(For example, a firmware update module must restart the system to invoke the new
software), and Secure Device Onboard must complete “on faith” that all is well.

Any error in a module must cause the entire Secure Device Onboard session to fail with an
error message.

#### Module Selection Using the Pre-ServiceInfo (PSI) Variable[^18]

[^18]: We intend to replace the PreServiceInfo variable with a more general
mechanism in future versions of Secure Device Onboard.

The “Pre-ServiceInfo” variable: TO2.GetNextDeviceServiceInfo.psi is a string
that starts the ServiceInfo handshake. The Psi variable encodes a
mini-ServiceInfo. The format of the “psi” variable is:

*modName1* **:** *modMsg1* **\~** *modVal1* **,** *modName2* **:** *modMsg2*
**\~** *modVal2* …

This is a comma-list of module-message-value triplets, where the module name is
delimited with a colon (‘:’), and the message is delimited from the value with a
tilde (‘\~’).

The moduleData is used when the Secure Device Onboard Device must perform specific
Owner-driven operations before the Device ServiceInfo. For example, a TPM module
might need the number of key pairs to allocate.

The purpose of the Pre-ServiceInfo (PSI) variable is to allow the Secure Device Onboard
Owner to require specific action from the Secure Device Onboard Device before it sends the
Device ServiceInfo. This might be required, for example, if the Secure Device Onboard Owner
needed the Secure Device Onboard Device to allocate a specific number of keys or other
resources, then offer them to the Secure Device Onboard Owner. This choice might imply a
preference from the Owner to the Device as well.

If the Secure Device Onboard Owner has no such requirement, it need not generate the
Pre-ServiceInfo. For example, a constrained Secure Device Onboard Device may always generate
the same Device ServiceInfo. In this case, the Pre-ServiceInfo has no function.

Module information in the PSI may be used to indicate a preference for one
module over another. For example, the Secure Device Onboard Owner may indicate that a TPM2
module is preferred over a TPM1.2 module, even if both are supported by the
Owner and the Device.

Consider the following psi variable:

tpm-1.2:pref\~tpm-2;tpm-2:keys\~3,tpm-2:type\~ecc,

>   tpm-1.2:keys\~3,tpm-1.2:type\~rsa

This might indicate support for: a tpm module at version 2 that has an argument
indicating 3 RSA key-pairs are needed, the same module at version 1.2 also
needing 3 key-pairs (but ECC); and a preference for module tpm-2 over tpm-1.2
(the “pref” message).

Following this example, an Secure Device Onboard Device that supports TPM 1.2 can enable the
appropriate module and allocate 3 RSA key pairs, and another Secure Device Onboard Device
with both TPM 1.2 and TPM 2.0 capability can select TPM 2.0 and allocate 3 ECC
key pairs. In each case, the Device ServiceInfo is used to activate a particular
module.

Constrained Secure Device Onboard implementations do not need to parse Pre-ServiceInfo,
unless they need to access moduleData. Specifically, a constrained Secure Device Onboard
Device that always allocates one key pair does not need to scan the “psi”
variable.

#### Module Activation in Device ServiceInfo

A module on the Secure Device Onboard Device indicates its availability to the Secure Device Onboard
Owner by sending the **active** message with value 1 (true):

*In Device ServiceInfo:* … "*modName***:active**":"**1**", …

The **active** message must precede all other messages sent by a given module.

An Secure Device Onboard Owner may only send messages to a module that has sent an
**active** message. Messages to non-existent or non-activated modules cause the
Secure Device Onboard session to be terminated with an error message.

An Secure Device Onboard Owner may refuse the activation of a module by sending a
de-activating **active** message:

*In Owner ServiceInfo:* … "*modName***:active**":"**0**", …

If the Secure Device Onboard Owner sends a de-activate message, it may not send any other
messages to this module in this Secure Device Onboard session. The Secure Device Onboard Owner must
de-activate all modules that it does not intend to use.

For example, a constrained Secure Device Onboard Device may implement a management module
and a firmware update module. It activates both modules, sending the current
firmware version as a parameter of firmware update. The Secure Device Onboard Owner can
decide either to accept this version of firmware and de-activate the firmware
update module, or decide to update the firmware and de-activate the management
module.


#### Secure Device Onboard Version 1.0 Key-Values

Version 1.0 of Secure Device Onboard implemented a different set of key-value pairs. We have
elected to replace this mechanism with the current one.

Implementers who need to support the previous Device- and Owner-ServiceInfo
key-value pairs should contact the Secure Device Onboard Enablement team at Intel.

#### Examples

In the following examples, spaces are provided for clarity, and fragments of
ServiceInfo are presented.

##### Expressing Values in Different Encodings

… "mymod:options","foo,bar"…

… "mymod:options","Zm9v,YmFy" …

… "mymod:options","Zm9vLGJhcg==" …

These 3 example each defines a message “options” with value “foo,bar”. The first
gives the value in printable ASCII, the second in a list of base64 values, the
third as a list completely encoded in base64.

Which value is correct depends on the implementation of “mymod”.

##### Hypothetical File transfer (Owner ServiceInfo)

"binaryfile:name","myfile.tmp",

"binaryfile:length","1234",

"binaryfile:data001","—*base64-data-512-bytes*—",

"binaryfile:data002","—*base64-data*—*512-bytes*—",

"binaryfile:data003","—*base64-data-210-bytes*—",

"binaryfile:sha-384","—*base64-data*—*48-bytes*—"

In this example, a “binary file” module allows a file to be downloaded using the
Secure Device Onboard secure channel. The data002 and data003 variables need to be in
separate ServiceInfo messages to keep message sizes within spec. Base64 encoding
is used to allow the module to generate a binary file. The last message allows
the file transfer to be verified after it is stored in the filesystem, as an
added integrity check.

Another way to accomplish file transfer would be to use an external HTTP
connection. For example:

"wget:filename":"myfile.tmp",

"wget:url":"http://myhost/myfile.tmp",

"wget:sha-384":"—*base64-data*—*48-bytes*—"

In this case, the file is transferred using a separate connection, perhaps at OS
level. If the file is confidential, ‘https:’ could be used instead of ‘http:’.

Both these techniques are valid in Secure Device Onboard, and represent two sides of a
trade-off. Using the Secure Device Onboard channel, a small file can be transferred without
needing a parallel network connection (see section [§](..//#limitation-of-round-trip-times) for limitations on
the Secure Device Onboard channel size). However, the same file might be transferred much
faster using an optimized HTTP implementation, and might not require the
confidentiality built into Secure Device Onboard (For example, the file contents might be posted on
a public Internet site). Table ‎23 discusses the trade-off.

Secure Device Onboard is tuned to provide access to the Secure Device Onboard server using HTTP or
HTTPS. Since Secure Device Onboard may run in a proxy environment created by the Secure Device Onboard
Installer Tool (see section [§](../introduction/#introduction)), other protocols should be used only when
both the Secure Device Onboard Device, Secure Device Onboard Owner and the installation network are
known to support them. Otherwise, stick with HTTP/HTTPS.


<table>
    <caption>Table 23 -  Comparison of Transferring a File Using Secure Device Onboard Channel or
Independent Channel</caption>
    <thead>
        <tr>
            <th>File Transfer Using Secure Device Onboard Channel</th>
            <th>File Transfer Using HTTP Mechanism</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Performance based on Secure Device Onboard protocol (slow for file large data)</td>
            <td>Performance based on HTTP or HTTPS, designed for streaming large amounts of data. Second stream required.</td>
        </tr>
        <tr>
            <td>Can download large amounts of bulk data or programs (limited to the number of ServiceInfo iterations)</td>
            <td>Can download arbitrary amounts of bulk data or programs</td>
        </tr>
        <tr>
            <td>Data can be stored in a file</td>
            <td>Data can be stored in a file</td>
        </tr>
        <tr>
            <td>Data can be executed as a program</td>
            <td>Data can be executed as a program</td>
        </tr>
        <tr>
            <td>Data is encrypted using Secure Device Onboard channel</td>
            <td>Data is encrypted only if HTTPS is used.</td>
        </tr>
        <tr>
            <td>Data is verified using Secure Device Onboard channel</td>
            <td>Data can verified by the module if a hash of contents (For example, SHA-384) is included</td>
        </tr>
    </tbody>
</table>

##### Hypothetical Direct Code Execution

"code:architecture","x86_64",

"code:length":"512",

"code:machinecode001","—*base64-data-512-bytes*—",

In this example, a module permits loading and executing machine code (this might
be needed on a MCU). Obviously, this requires a high degree of trust in the
Secure Device Onboard implementation, and perhaps an ability to execute code in a sandbox.

#### Implementation Notes

This section is not logically part of the Secure Device Onboard specification.

An Secure Device Onboard implementation may implement ServiceInfo in a variety of ways. It
is recommended that Secure Device Onboard implementations create a ServiceInfo interface on
both Device and Owner side, that allows an easy plug-in mechanism.

On the Owner side, a dynamic plug-in mechanism may by easier to maintain.

On the Device side, a statically linked or compiled mechanism may be required
due to system constraints. However, a more capable Device that runs Linux OS
might be able to implement a flexible scripted mechanism similar to **init.d**.

As stated above, constrained Devices do not need to implement pre-ServiceInfo,
unless they actually present choices of module, or except if they need to scan
for a modData parameter.

In a given implementation it is possible to process the ServiceInfo variables as
they arrive or in a batch. However, the order of interpretation of messages must
be preserved.

