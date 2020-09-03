# Appendix B: Device Provisioning with ECDSA

The following procedure is used to initialize the Secure Device Onboard
Device key and certificate, before the Secure Device Onboard Device Initialize
(DI) protocol is run:

-   An ECDSA key pair is generated, and a Certificate Signing Request (CSR) is
    signed with the new private key.

    -   The recommended way to do this is to generate the ECDSA key pair and the
        signed CSR inside the Secure Device Onboard device.

    -   If an appropriate security level is possible in device manufacture, it
        is acceptable for an ODM to generate the key pair outside the Secure Device Onboard
        Device, generate its own CSR, program the Secure Device Onboard Device with the
        private key, then discard its copy of the ECDSA private key.

-   The CSR is submitted to a Certificate Authority trusted by the ODM to create
    a Device certificate and certificate chain.

    -   The device certificate should not expire unless the ODM has a reason for
        Secure Device Onboard to be performed before a certain date.

-   The Device private key shall be stored with CAI (Confidentiality,
    Availability, Integrity) protection in the Device TEE that is performing
    Secure Device Onboard.

-   The certificate chain is attached to the Ownership Voucher, as described in
    section [§](../detailed-protocol-description/#persisted-messages) : Persisted Messages Type 3 for more info

The Ownership Voucher HMAC, passed in the DI protocol, references the initial
Device Certificate. This means that the ECDSA key and certificate must be
programmed before the Device Initialize protocol is run. The Manufacturer is
trusted to match the Device certificate information to the required DI protocol
fields. Subsequent to this, the Ownership Voucher HMAC (OwnershipProxy.hmac) is
used to detect if the Device Certificate is changed in the supply chain.

