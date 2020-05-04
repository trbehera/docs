# Appendix C: Secure Device Onboard 1.1 Crypto Upgrade

In October of 2017, the Security Architecture Forum (Intel Internal forum)
(SAFE) Crypto Guidelines were upgraded, based on the potential threat of quantum
cryptography. This table indicates crypto levels to apply. Secure Device Onboard 1.0 and Secure Device Onboard 1.1
protocol spec use the left column, and the new crypto strength is given in the
“Future Crypto” column; it will be assigned a Secure Device Onboard release in the
future*.*

*The length of a release is generally outside the scope of this document;
however, implementers of Secure Device Onboard Service and Owner components must take care
to ensure that Devices using Secure Device Onboard 1.1 crypto must be supported by all Secure Device Onboard components for the life of the Secure Device Onboard 1.1 release.*

<table>
    <thead>
        <tr>
            <th>Category</th>
            <th>Secure Device Onboard 1.0 &amp; Secure Device Onboard 1.1</th>
            <th>Future Crypto (Enhanced Strength for Quantum Crypto)</th>
            <th>Comments</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Device HMAC</td>
            <td>HMAC-SHA-256 with HMAC secret of 128 bits</td>
            <td>HMAC-SHA-384 with HMAC secret of 512 bits</td>
            <td>Bay Trail grandfathered for SHA-256 <a href="../protocol-description/#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Hash of Owner Key</td>
            <td>SHA-256</td>
            <td>SHA-384</td>
            <td>(inside Device DI protocol) It is also permissible to store the entire Owner key. <a href="../protocol-description/#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Ownership Voucher (Owner attestation)</td>
            <td>RSA2048 Use RSA2048RESTR as public key type.</td>
            <td>RSA3072, except Bay Trail Use RSA_UR as public key type.</td>
            <td>Bay Trail grandfathered for RSA2048RESTR option. <a href="../protocol-description/#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Ownership Voucher (Owner attestation)</td>
            <td>ECDSA NIST P-256</td>
            <td>ECDSA NIST P-384</td>
            <td>P-384 not supported by some crypto chips; it may be possible to grandfather them in. <a href="../protocol-description/index.html#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Ownership Voucher</td>
            <td>SHA-256</td>
            <td>SHA-384</td>
            <td>Bay Trail grandfathered for SHA-256. <a href="../protocol-description/#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Device attestation EPID1.0, 1.1, 2.0</td>
            <td>Per platform</td>
            <td>No change</td>
            <td>Intel<sup>®</sup> EPID is under evaluation by SAFE; Later versions of Intel<sup>®</sup> EPID are being developed by labs. <a href="../protocol-data-types/#intel-enhanced-privacy-id-intel-epid-signatures-overview">See §</a></td>
        <tr>
            <td>Ownership Voucher</td>
            <td>SHA-256</td>
            <td>SHA-384</td>
            <td>Bay Trail grandfathered for SHA-256. <a href="../protocol-description/#building-the-ownership-credential-ownership-voucher">See §</a></td>
        </tr>
        <tr>
            <td>Device attestation ECDSA</td>
            <td>ECDSA NIST P-256 (with SHA-256 hash) &amp; ECDSA NIST P-384 (with SHA-384 hash)</td>
            <td>ECDSA NIST P-384 (with SHA-384 hash)</td>
            <td>This is the device attestation key built into the device. <a href="../protocol-data-types/#ecdsa-nist-p-256-and-ecdsa-nist-p-384-signatures">See §</a></td>
        </tr>
        <tr>
            <td>Key Exchange, DH</td>
            <td>Id14 (2048-bit modulus) a, b are 256 bits each</td>
            <td>Id15 (3072 bit modulus) a, b are 768 bits each</td>
            <td>New modulus available from same RFC as old, see refs. Increased entropy needed for SVK, SEK. <a href="../protocol-description/index.html#diffie-hellman-key-exchange-protocol">See §</a></td>
        </tr>
        <tr>
            <td>Key Exchange, Asymmetric</td>
            <td>RSA2048RESTR RSA-OAEP-MGF-SHA256 Device, Owner Random of 256 bits</td>
            <td>RSA_UR, 3072 bits RSA-OAEP-MGF-SHA256 Device, Owner Random of 768 bits</td>
            <td>*SHA256* may be used here as mask generation function for RSA-OAEP. Larger D&amp;O Randoms required for SVK, SEK. <a href="../protocol-description/#asymmetric-key-exchange-protocol">See §</a></td>
        </tr>
        <tr>
            <td>Key Exchange, ECDH</td>
            <td>ECC NIST P-256 or NIST P-384, keys used once only Device, Owner random of 128 bits</td>
            <td>ECC NIST P-384, keys used once only Device, Owner random of 384 bits</td>
            <td>Larger D&amp;O Randoms required for SVK, SEK. <a href="../protocol-description/#ecdh-key-exchange-protocol">See §</a></td>
        </tr>
        <tr>
            <td>Key Exchange, ECDH, LEGACY</td>
            <td>ECC NIST P-256, keys used once only Device, Owner random of 128 bits</td>
            <td>ECC NIST P-256, keys used once only Device, Owner random of 512 bits</td>
            <td>Note that this mode can only be used for legacy hardware, with approval from PSE. <a href="../protocol-description/index.html#ecdh-key-exchange-protocol">See §</a></td>
        </tr>
        <tr>
            <td>Key Derivation Function (*‎2.5.5.4*)</td>
            <td>SHA-256 based SEK, SVK entropy is 128 bits (SVK 256 bits, but with lower entropy)</td>
            <td>SHA-384 based SEK is 256 bits SVK is 512 bits</td>
            <td>Note changes to Device and Owner Randoms in key exchange algorithms to support this change. <a href="../protocol-description/index.html#key-derivation-function">See §</a></td>
        </tr>
        <tr>
            <td>SEK (Session Encryption Key)</td>
            <td>128 bits</td>
            <td>256 bits</td>
            <td>Matches TO2 Session Encryption modes. <a href="../protocol-description/#key-exchange-in-the-to2-protocol">See Table in §  </a>; also <a href="../data-transmission-persistence/#encrypted-message-body">see §</a></td>
        </tr>
        <tr>
            <td>SVK (Session Verification Key)</td>
            <td>256 bits, with 128 bits entropy</td>
            <td>512 bits</td>
            <td>Based on 512 bit state of SHA-384. <a href="../protocol-description/#key-exchange-in-the-to2-protocol">See Table in §  </a>; also <a href="../data-transmission-persistence/#encrypted-message-body">see §</a></td>
        </tr>
        <tr>
            <td>TO2 Session HMAC</td>
            <td>HMAC-SHA-256</td>
            <td>HMAC-SHA-384</td>
            <td>HMAC key is 512 bits. <a href="../protocol-description/#key-exchange-in-the-to2-protocol">See Table in §  </a>; also <a href="../data-transmission-persistence/#encrypted-message-body">see §</a></td>.
        </tr>
        <tr>
            <td>TO2 Session Encryption, counter mode</td>
            <td>AES-128/CTR IV 12 bytes, Counter is 4 bytes.</td>
            <td>AES-256/CTR IV is 12 bytes, Counter is 4 bytes</td>
            <td>Note session limits on TO2 protocol (<a href="../detailed-protocol-description/#limitation-of-round-trip-times">See §</a>).</td>
        </tr>
        <tr>
            <td>TO2 Session Encryption, CBC mode</td>
            <td>AES-128/CBC IV is 16 bytes</td>
            <td>AES-256/CBC IV is 16 bytes</td>
            <td>Note session limits on TO2 protocol (<a href="../detailed-protocol-description/#limitation-of-round-trip-times">See §</a>).</td>
        </tr>
        <tr>
            <td>TO2 Protocol Roundtrip Limit</td>
            <td>1M (1e6) rounds</td>
            <td>1M (1e6) rounds</td>
            <td>Required limit for AES/CTR mode. Increased limit is to permit small files to be transmitted in ServiceInfo, esp for MCUs. <a href="../detailed-protocol-description/#transfer-ownership-protocol-2">See §</a></td>
        </tr>
    </tbody>
</table>

