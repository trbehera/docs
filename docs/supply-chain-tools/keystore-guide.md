## Introduction

Secure Device Onboard enhances the out-of-the-box setup and provisioning experience for connected IoT devices.

The SDO reseller toolkit addresses the needs of resellers, manufacturers, and others in the supply chain to transfer ownership of SDO devices.

To learn more about SDO, see [Overview](#overview).

## Terminology

Refer to the [Secure Device Onboard Reference page](../reference.md).

## Reference Documents

Refer to the [Secure Device Onboard Reference page](../reference.md).


## Overview

The SDO supply chain tools (manufacturer and reseller toolkits) require a key pair to be used to extend the ownership voucher and in the case of elliptic-curve cryptography (ECC)-based devices, to extend the device certificate chain.

This key pair must be generated and be made available to the toolkits in either the key store file format or in the hardware key (secure key fob). The file option is typically used for development, test, and evaluation purposes while the hardware key must be used in a production environment.

Choose the appropriate method for your situation. This process must be done for each key type you need to support. Key types used in SDO devices include RSA256, ECC256, and ECC384. Depending on the type of devices you plan to support you will need to generate from 1 to 3 key types, each type resulting in a key store file and a set of keys.


## Step 1: Key Store File Initialization

Use the [p12init.sh](https://github.com/secure-device-onboard/supply-chain-tools/blob/master/scripts/p12init.sh) script provided along with the toolkit to generate the key store file. Before use, update the configuration in the file (For example, key store password and certificate validity period).

The script can be run directly on a Linux\* OS. For Windows\* OS, you will need to install the Cygwin\* DLL from <https://www.cygwin.com/> and run the script from a Cygwin\* shell. If running with Cygwin\*, be sure to add Java\* to your path variable. For example, in a Cygwin\* command window:

Export Path=\$PATH:"/cygdrive/C/program files/java/jdk-11/bin"

You should now have a key store file that can be used by the toolkit. You will need to provide the name of this file and the password you used here to the toolkit web service later.

## Step 2: Hardware Key Initialization

1.  Sensitive cryptographic material is handled during the initialization of a hardware key. For this reason, the system used here needs to be secure. the Secure Device onboard project strongly recommends an air-gap system as detailed in [Step 3: Prepare an Air-Gap System](#step-3-prepare-an-air-gap-system).

2.  The key pair backup must be stored securely where only authorized personnel have access to. Should this key pair be compromised, all active devices initialized with this key pair may be required to be re-initialized with SDO.

## Step 3: Prepare an Air-Gap System 

The air-gap system is a machine that is as follows:

-   Has no network interface enabled (wired, wireless, or other such as Bluetooth<sup>®</sup>).

-   Is free from any viruses or malware.

-   Can run the OpenSSL\* toolkit.

-   Has a USB\* interface.

Before the system is taken off the network, download the Yubikey* smart card driver and tools (<https://www.yubico.com/products/services-software/download/smart-card-drivers-tools/>).

## Step 4: Setup the Secure Key Fob

Use the **ykinit.sh** script provided along with the toolkit to set up the secure key fob (YubiKey\* security key). You will need one YubiKey\* security key for each key type you wish to support.

1.  Copy this script to the system prepared in [Step 3: Prepare an Air-Gap System](#step-3-prepare-an-air-gap-system).

2.  Insert a YubiKey* security key into the system.

3.  Run the script and select the desired key type. (If running on Windows\* OS, you will need to install the Cygwin\* to run the script).

4.  Make a backup of the output files produced by the script. A backup copy of the key material is useful if any of your key fobs fail. You can use this backup to create a replacement key fob. Be sure to store this copy in a physically secure location that is only accessible by authorized personnel.

5.  Use the YubiKey manager to change the pin code on the YubiKey* security key. You will need to specify this pin code to the application later. Example: ykman piv -change-pin

6.  For each additional key type (For example, EC256, EC384, RSA2048), repeat steps 2 to 5. One Yubikey* security key is required for each key type.

## Step 5: Setting up a Secure Key Fob from an Existing Key and Cert

Follow the steps in this section to set up a Yubikey\* security key from the existing key material. These steps produce a backup Yubikey\* security key or an additional Yubikey\* security key for use with multiple Toolkit installations where all Toolkits need to use the same key pair.

Locate the copies of the crt (refer to the definition at the end of this section) and key files from the original setup process. The following are steps to set up each Yubikey* security key required:

1.  ykman piv reset

2.  ykman piv import-key \--management-key \<mkey\> 9a \<key file\>

3.  ykman piv import-certificate \--management-key \<mkey\> 9a \<crt file\>

4.  Use the YubiKey* manager to change the pin code on the YubiKey\* security key: ykman piv change-pin

where mkey is the Yubikey* management key (default value is 010203040506070801020304050607080102030405060708)

key file = filename containing the private key

crt file = filename containing the certificate
