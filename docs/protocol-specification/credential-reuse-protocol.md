# Credential Reuse Protocol

Credential Reuse protocol allows devices to reuse the Ownership Credential
across multiple onboardings. The intended use case for this protocol is to
support demos and testing scenarios where the onboarding can be run repeatedly
and quickly without having to change the Ownership Voucher or resetting the
system after each onboarding. Since credential reuse can permit the previous
Owner unlimited access to the device, it is NOT recommended for use in the
normal device supply chain.

Credential reuse is selected by the Owner, and accepted or rejected by the
device. We anticipate that new devices will always allow credential reuse, but
some legacy devices do not support it. However, we can envision devices for high
security applications which might reject credential reuse.

In normal credential use, the Owner changes the Ownership Credential in
TO2.SetupDevice, which also creates a new Ownership Voucher. At the end of a
successful TO2 protocol, the device deactivates Secure Device Onboard and its state needs to
be changed to ReadyN using the Resale protocol in order to reactivate onboarding
process on next boot. The next onboarding uses the new Ownership Voucher.

For credential reuse, the TO2 protocol supports a special case which indicates
to the device not to change the Ownership Credential in TO2.SetupDevice. The
device still runs the complete TO2 protocol to the end but does not deactivate
Secure Device Onboard at the end of the protocol.

The Credential Reuse protocol is as follows:

In TO2.SetupDevice:

-   **If** TO2.SetupDevice.noh.bo.g3 == TO2.ProveOPHdr.bo.oh.g \# GUID same as
    previous,

    **and** TO2.SetupDevice.noh.bo.r3[0] == TO2.ProveOPHdr.bo.oh.r \# RendezvousInfo same as previous,

    **and** TO2.SetupDevice.pk == Owner’s current public key \# public key in the last entry of Ownership Voucher,

    **and** TO2.SetupDevice.sig is a valid signature

-   **Then**

    -   Device does not update the Ownership Credential,

    -   **and** Device does not internally change the HMAC,

    -   **and** in TO2.Done message, devices responds with TO2.Done.hmac equal
        to the ASCII string “=” (i.e., hmac.length = 1 and value = 0x3d;
        hmac.hashtype = 0).

Note that an HMAC with one byte (hmac.length=1) is not a legal HMAC. The special
value for TO2.Done.hmac is used to indicate to the Owner that device supports
Credential Reuse protocol, and that the HMAC is not changed.

Devices which do not support credential reuse generate a new HMAC and return its
value, a valid HMAC, in TO2.Done.hmac. The Owner can differentiate whether the
device supports Credential Reuse based on the HMAC value.


