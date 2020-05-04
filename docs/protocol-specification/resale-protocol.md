# Resale Protocol

After the transfer of ownership completes (For example, the TO2 Protocol finishes), the
Device switches to an idle state (Device Secure Device Onboard State = IDLE), which inhibits the
device’s software from running Secure Device Onboard. See
the Secure Device Onboard Architectural Specification for a
description of Secure Device Onboard Device States.

A device implementation might also stop a thread or process from running to
achieve the same effect, perhaps freeing resources for Device operation. In the
case of a MCU-based implementation, the Secure Device Onboard code might only be able to run
when external software calls a specific entry point for it.

The Owner may use System or OS level commands to re-enable Secure Device Onboard for a new
transfer of Ownership. In complex operating systems, this can be done by setting
the device to the ReadyN (Transfer Ready) state; other system level measures
must be arranged on a per-implementation basis.

In the TO2 Protocol (section [§](../detailed-protocol-description/#transfer-ownership-protocol-2)), the Secure Device Onboard software in the Device TEE stores new
credentials that are only known to the Owner. How the device info is updated is
described in section [§](../detailed-protocol-description/#to2setupdevice-type-47), which describes the
TO2.SetupDevice message. Please note that the public
key stored in the device is updated to the “Owner2” key, a key that is separate
from the Owner key in the original Ownership Voucher. This is to prevent any way
of correlating the original Ownership Voucher from the one being generated for
resale in the TO2 Protocol.

Subsequently, in the TO2.Done message, the Device transfers
to the Owner the HMAC of the stored device credentials. This HMAC is used by the
Owner exactly as the HMAC supplied to the ODM in the
DI.SetHMAC message is used, to create a new Ownership
Voucher.

Resale, then, involves the following steps:

1.  The Device is reconditioned to remove all run-time changes and brought back
    to a factory state. This includes removing any secrets, except for the Secure Device Onboard credentials from the TO2 Protocol.

2.  The Device is instructed to transition to a ReadyN state, and any other
    actions needed to enable to Secure Device Onboard Device software to run are performed.

3.  At this point, the Device is ready to transfer ownership. It may be powered
    down, shipped, and re-installed in a new location. Note that the GUID in the
    Device has changed since it was manufactured.

4.  The Owner transfers the Ownership Voucher to the Manufacturing tool.

5.  The Manufacturing tool signs the Ownership Voucher to the next Owner and
    sends the updated Ownership Voucher on.

6.  Eventually, the new Owner receives the new Ownership Voucher, with a
    signature chain of one or more signatures. The new Owner initiates the TO0
    Protocol.

7.  Eventually, the Device is installed in its new location. The Device starts
    to run the TO1 Protocol to determine the new Owner’s Internet location.

8.  The Device segues from TO1 to TO2 Protocol to transfer ownership to the new
    Owner.

9.  As a side effect, yet another Ownership Voucher is created for the new
    Owner, and so on.

It may be that, when resale time comes, the Owner wishes to change the
rendezvous information that is stored in the Device TEE. This may be
accomplished by performing a transfer of ownership (using the TO2 Protocol) from
the Owner to itself, allowing replacement of the credentials in the
TO2.SetupDevice message.

## Secure Device Onboard Devices that Do Not Support Resale

A device may, at its option, implement only a limited number of Secure Device Onboard 
transfers of ownership. There are various reasons for this:

-   Each transfer might consume some OTP memory, and the total amount is
    limited.

-   A device is intended to be discarded after its first Ownership Transfer.

-   The ability to use Secure Device Onboard again on a Device might be
    thought of as an attack vector to disable or even steal the device as the
    latter requires compromising both the Device and its current Owner.

In this case, the Device must be careful to disable Secure Device Onboard
software after the initial transfer of ownership succeeds. This can be
accomplished using the Device State: PD (Permanently Disabled). If Secure Device Onboard is
disabled for security reasons, it is best also to destroy any credentials in the
Device or to prevent the code from running using ad hoc mechanisms, such as
uninstalling the application code.

As described in section [§](../detailed-protocol-description/#to2done-type-50), the TO2.Done message can also inform the Owner
of the Device’s inability to perform resale by transmitting zero length HMAC.

## Secure Device Onboard Owner that Does Not Support Resale

An Owner may elect not to support the resale capability, even if the underlying
Device is capable of doing so. The Owner is still required to provide new
credentials for the Device in the [TO2.SetupDevice]
message. The Owner should then discard the credentials in a manner that will
ensure that neither the Owner itself nor any malicious party can ever obtain
them. This involves:

-   The Owner must ensure the security of the Owner2 private key such as
    discarding the key.

-   The Owner must delete the HMAC received from the Device.

-   The Owner must not extend the Ownership Voucher before deciding to discard
    the key or HMAC.

