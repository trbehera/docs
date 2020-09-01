# Introduction

This document specifies the protocol interactions and message formats for the 
Secure Device Onboard protocols. Secure Device Onboard is a device 
provisioning scheme, sometimes called device “onboarding.”

Device provisioning is the process of installing secrets and network addresses
into a device so that the device and its network-based manager are able to
connect using a trusted connection.  For example, they may share a secret or
have trusted PKI credentials.  

Device provisioning causes a new entity to own the device.  This requires a
strong degree of trust, since the device owner must at least provision
communications credentials into a Device, and probably needs complete control
over the device operating system and software.  For example, the device owner
must patch security vulnerabilities, install or update drivers, query hardware
devices, program actuators, and others.  A given kind of device may restrict what an
owner can do to it using Secure Device Onboard, trading off the risk of
remote network compromise with the cost of perhaps needing to physically access
the device to do some restricted configuration.

For the purposes of Secure Device Onboard, we assume that the device initiates a connection
to the manager, and not the reverse.  This is common industry practice, and has
the advantages that:

1.  The IP address of the manager is less likely to change than the IP address
    of the device.

2.  The traffic outbound through network firewalls tends to be more permissive
    than the reverse, increasing the *likelihood* that the device to manager
    connection will not need special firewall policy.

In Secure Device Onboard, device physical installation is kept separate from device keying
and remote control.  

Secure Device Onboard works by establishing the ownership of a device during manufacturing,
then tracking the transfers of ownership of the device until it is finally
provisioned and put into service.  In this way, the device provisioning problem
can be thought of as a device “transfer of ownership” problem. In Secure Device Onboard, we
assume that this factory-time configuration is the only configuration done to
the device.  Other steps taken outside the device make it possible for the
device to use this configuration to find and establish trust with its owner when
it is first powered on after manufacturing.  

Between when the device is manufactured and when it is first powered on and
given access to the Internet, the device may transfer ownership multiple times. 
A structured digital document, called an Ownership Voucher, is used to transfer
digital ownership credentials from owner to owner without the need to power on
the device.

When the device is first plugged in, the Secure Device Onboard protocols are invoked,
causing the device to access the Secure Device Onboard Rendezvous Server.  By protocol
cooperation between the device, the Rendezvous server, and the new owner, the
device and new owner are able to prove themselves to each other, sufficient to
allow the new owner to establish new cryptographic control of the device.  When
this process is finished, the device is equipped with:

-   The Internet address and public key of its manager

-   A random number that may be used as shared secret

-   Optionally, a key pair whose public key is in a certificate signed by a
    trusted party of its manager

This is expected to be a strong and flexible set of credentials, enough to
satisfy most IoT device to manager needs. For example, this permits a 2-way
authenticated Transport Layer Security (TLS) connection from device to manager.

In the case where other credentials are needed, a TLS connection with the
provisioned credentials should suffice to allow the manager to provide different
ones.

In our current model, we assume the device has access to the Internet when it is
first powered on. In some environments, this will require that a
second *Secure Device Onboard Installer Tool* be used to provide a temporary, perhaps restricted, Internet
connection, sufficient to allow Secure Device Onboard to proceed.

During manufacturing, an Secure Device Onboard equipped device is ideally configured with:

-   A processor containing:

    -   A Trusted Execution Environment (TEE), with cryptographic hardware root of
    trust, based on either Intel<sup>®</sup> EPID or Elliptic Curve Digital Signature
    Algorithm (ECDSA)

    -   An Secure Device Onboard application that runs in the processor’s TEE that maintains and
    operates on device credentials

-   A set of device ownership credentials, accessible only within the TEE:

    -   Rendezvous information for determining the current owner of the device

    -   Hash of a public key to form the base of a chain of signatures, referred to
    as the Ownership Voucher

    -   Other credentials, please see section [§](../detailed-protocol-description/#disetcredentials-type-11) for more information.  

Secure Device Onboard may be deployed in other environments, perhaps with less expectation
of security and tamper resistance. These include:

-   A microcontroller unit (MCU), perhaps with a hardware root of trust, where
    the entire system image is considered to be a single trusted object

-   An OS daemon, with keys sealed by a Trusted Platform Module (TPM) or in the
    filesystem

The remainder of this document presumes the ideal environment, as described
previously.

## Transmitted Protocol Version

The current protocol version of Secure Device Onboard is **1.13**

Every message of the transmitted protocol for Secure Device Onboard specifies a
**protocol version**.  This version indicates the compatibility of the
protocol being transmitted and received.  The actual number of the
protocol version is a major version and a minor version, expressed in
this document with a period character ('.') between them.

The specification version may be chosen for the convenience of the
public.  The protocol version changes for technical reasons, and may
or may not change for a given change in specification.  The protocol
version and specification version are different values, although a
given specification must map to a given protocol version.

The receiving party of a message can use the protocol version to
verify:

-   That the version is supported by the receiver

-   That the version is the same as with previous received messages
    in the same protocol transaction

-   Whether the receiver needs to invoke a backwards compatibility
    option.  Since Secure Device Onboard allows the Device to choose any supported
    version of the protocol, this applies to the Owner or Rendezvous.

## Privacy Concerns

Secure Device Onboard has a number of protocol features to preserve the privacy of a
device’s progress from manufacturing to ownership, to resale or decommissioning.

All keys exposed by protocol entities are created only for the
purposes of Secure Device Onboard, and can be used temporarily, and new ones generated frequently to keep them
from being used as to profile a manufacturer, a device, or a device
owner/operator.

The Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) can be used to prove the ability to
take ownership without identifying the device to the Rendezvous server or to
anyone monitoring Internet traffic at the Rendezvous server. Intel® EPID may also
be used to prove manufacturer and model number without identifying device.

Transfer Ownership protocol 2 (TO2), replaces all keys and identifiers in the
device, except the root of trust, so that no record of previous ownership chain
exists. In case of Intel<sup>®</sup> EPID, it is not necessary to replace the Intel<sup>®</sup> EPID key
for based on a privacy concern.

IP addresses can be allocated dynamically by the device owner. This does not
prevent a determined adversary from using IP addresses to trace devices, but can
raise the bar against more casual attempts to trace devices from outside to
inside an organization.

## Intel<sup>®</sup> Enhanced Privacy ID (Intel<sup>®</sup> EPID) Attributes

As Intel has acted to enable other companies to use Intel<sup>®</sup> EPID2 for a variety of
devices and applications, a new Intel<sup>®</sup> EPID2 attributes feature has been added by
the Intel Key Generation Facility
([iKGF](https://contentprotection.intel.com/)), an independent key issuing
authority. Intel<sup>®</sup> EPID attributes are identifiers associated with an Intel<sup>®</sup> EPID
group, and reflected in the group certificate. A common set of attributes is
also represented in the 128-bit EPID2 Group ID.

Group ID attributes are important because Intel<sup>®</sup> EPID does not identify a device
except as the member of a group. Thus, an Intel<sup>®</sup> EPID signature is only as
trustworthy as the least trustworthy member of the group, implying that—for best
results—all members of the group should be equally trustworthy.

The group ID attribute allows an Intel<sup>®</sup> EPID user to work with Intel Key
Generation Facility (iKGF) so that all devices in a given Intel<sup>®</sup> EPID group have
the same security posture (For example, the same binary or the same source code base).
The iKGF can work with the device builder to allocate multiple EPID groups from
within a single “group address space” so that a given product type can be
detected by a 128-bit mask-and-compare operation.

The Owner is responsible for maintaining a list of Intel<sup>®</sup> EPID attribute
mask-and-compare sets for each kind of device that it is provisioning. When an
Intel<sup>®</sup> EPID signature appears at the Owner (in the TO2 Protocol):

-   The Group ID is compared against the Intel<sup>®</sup> EPID Attribute mask-and-compare
    sets to determine that this device is supported by this Owner (if not, the
    signature is rejected)

-   The Group is checked for revocation (if so, the signature is rejected)

-   The signature is verified against the group public key (if it fails to
    verify, the signature is rejected)

We envision that Intel<sup>®</sup> EPID attribute mask-and-compare sets are included by
manufacturers as part of their product documentation.

## Secure Device Onboard Terminologies

SIGMA is a protocol that allows an Intel<sup>®</sup> EPID device to establish a trusted
connection with a known entity, based on the provisioning of a Public Key
Infrastructure (PKI) certificate on the trusted entity’s server. This is similar
to the operation performed by the Transfer Ownership Protocol 2 (TO2).

In Secure Device Onboard, the trust of the Owner’s server is derived from the manufacturer’s
credentials (its public key), through the Ownership Voucher, to the Owner
key.[^1] Trust in Secure Device Onboard follows the supply chain entities, and not an
outside single root of trust. In particular, Secure Device Onboard specifically does not
require server belonging to the Owner of a device to have a certificate that is
rooted in any particular authority chosen by Intel.

[^1]: The Owner Key is the key signed in the last entry of the Ownership
Voucher. The corresponding private key is held by the current owner of the
Device. See section [§](../protocol-description/#the-ownership-voucher).

While this makes SIGMA itself inappropriate for Secure Device Onboard, the concepts of SIGMA
are incorporated in the TO2 protocol. For example, Secure Device Onboard uses a similar key
exchange and a Session Verification Key (SVK)/Session Encryption Key (SEK)
mechanism to create an encrypted channel.

Refer to the *Secure Device Onboard Glossary* document.

## Secure Device Onboard Transport Interfaces

Figure ‎1 describes the way in which Secure Device Onboard data is transported. Secure Device Onboard
protocols are defined in terms of an Secure Device Onboard message layer (section [§](../detailed-protocol-description/#detailed-protocol-description)) and
an encapsulation of these messages for transport to Secure Device Onboard network entities (section [§](../data-transmission-persistence/#data-transmission-persistence)).

Secure Device Onboard Devices may be either natively IP-based or non-IP-based. In the case
of Secure Device Onboard Devices which are natively connected to
an IP network, the Secure Device Onboard Device is capable of connecting directly to the Secure Device Onboard Owner or Secure Device Onboard
Rendezvous server. In some cases, an Secure Device Onboard Installer Tool is needed to allow
these devices to connect to the IP network. The Secure Device Onboard Installer Tool is a
HTTP/HTTPS Proxy, with security set up to allow only Secure Device Onboard protocols to
authorized Secure Device Onboard Rendezvous and Secure Device Onboard Owners.

Secure Device Onboard Devices which are not capable of IP protocols can still use Secure Device Onboard
by tunneling the Secure Device Onboard Message Layer across a reliable non-IP connection.
Secure Device Onboard messages implement authentication, integrity, and confidentiality
mechanisms, so any reliable transport is acceptable.

The Secure Device Onboard message layer also permits Secure Device Onboard to be implemented end-to-end
in a co-processor or trusted execution environment. However, security mechanisms
must be provided to allow credentials provisioned by Secure Device Onboard to be copied to
where they are needed. For example, if Secure Device Onboard is used to provision a
symmetric secret into a co-processor, but the secret is *used* in the main
processor, there needs to be a mechanism to preserve the confidentiality and
integrity of the secret when it is transmitted between the co-processor and
main-processor. This mechanism is outside the scope of this specification.



<figure id=fig-transportInterfaces>
<!--    <img src="../img/7c7b99cfa129cb06ef415793979d8255.emf" alt="Transport Interfaces" /> -->
    <img src="../img/fig-transport-interfaces.png" alt="Transport Interfaces">
    <figcaption>Figure 1 - Secure Device Onboard Transport Interfaces</figcaption>
</figure>

