ADILOS enables computer device users to gain access to resources. ADILOS uses a well-known cryptographic approach to perform a public-key exchange over an open channel.

ADILOS assumes an authorization process in which a user's public key is recorded in an ACL (access-control list) using an out-of-band communication. For a user this simply means displaying a public key so it can be entered in the ACL.

The reference implementation demonstrates data exchange using QR (quick read) optical codes scanned using high-definition cameras. This enables one to use a personal device such as a "smart" cellphone to obtain access.

ADILOS contains three subsystems:

* keymaster
* gatekeeper
* keymaster-gatekeeper communications interface ("kgprotocol")

## 1. keymaster

A reference implementation of a keymaster, for Android.

See [bitsanity / keymaster](https://github.com/bitsanity/keymaster)

## 2. gatekeeper

A reference implementation of a gatekeeper, for Raspbian Linux on Raspberry Pi 3.

See [bitsanity / gatekeeper](https://github.com/bitsanity/gatekeeper)

## 3. kgprotocol

Let/note:

1. **Curve** is the Elliptic Curve well known to cryptographers as __secp256k1__
2. **Signature** is an Elliptic Curve Digital Signature Algorithm (ECDSA) digital signature, generated using the Curve
  * 32 bytes (256-bits) in length
3. **Verify** means use ECDSA to verify a signature, using the Curve
4. **Private Key** is a 32-byte (256-bit) unsigned big integer initialized with a random value
5. **Public Key** is a value created by correctly applying a private key to the Curve
  * 33 bytes if using compressed form of key:
    * 32 bytes containing the X-coordinate of the point on the Curve
    * leading 0x03 byte indicating compressed key
  * 65 bytes if uncompressed form:
    * 64 bytes for the X,Y coord pair
    * leading 0x04 byte indicating uncompressed key
6. A **Challenge** consists of two parts concatenated and converted to Base64:
  * a gatekeeper's public key
  * a signature of the public key using the corresponding private key
7. A **Response** consists of two parts concatenated and converted to Base64:
  * public key selected by user (33 or 65 bytes)
  * signature of the message part (the gatekeeper's generated public key) of a Challenge, digitally signed with the user's selected key (32 bytes)

### 3.1 Expectations for any **keymaster**

* Must work equally well with any kgprotocol-compliant gatekeeper
* Secure the app and its data from unauthorized use (e.g. require PIN/passphrase on start)

### 3.2 Expectations for any **gatekeeper**

* Must work equally well with any kgprotocol-compliant keymaster
* May maintain an ACL (access control list) of authorized public keys
* May maintain relationships between a private key and other user-provided information
* Must generate a new, random key for each Challenge
* Must generate a new Challenge on start and after receipt of any cryptographically-valid Response

## kgprotocol Illustration

### Key Registration

| Ref. | keymaster Action | gatekeeper Action |
|:-----|:-----------------|:------------------|
| 1    | generate new key | |
| 2    | assign unique name to new key | |
| 3    | encrypt and store key name and value | |
| 4    | select key to register | |
| 5    | show public key (e.g. QR code) | |
| 6    | | scan/decode user-provided public key value |
| 7    | | store public key value in ACL |
| 5    | | optional: record other user information necessary for access control |

### Normal Operation

| Ref. | keymaster action | gatekeeper action |
|:-----|:-----------------|:------------------|
| 1    | | generate new key pair |
| 2    | | generate Challenge using new key pair |
| 3    | | encode and display Challenge openly |
| 4    | | scan for Response |
| 5    | approach gatekeeper | |
| 6    | select public key to use | |
| 7    | scan Challenge | |
| 8    | verify the challenge (check digital signature) | |
| 9    | if challenge valid, generate Response using selected key | |
| 10    | encode and display Response | |
| 10   | show Response to gatekeeper | |
| 11   | | scan and decode Response |
| 12   | | verify response (check digital signature) | |
| 13   | | parse public key from Response |
| 14   | | look up public key in ACL |
| 15   | | depending on result of lookup, allow or deny access |
| 16   | | return to step 1 |

## Expectations for ADILOS-compliant implementations

### Technical:

* Support __only__ the Curve for key generation, signing and verifying
* Adhere to __only__ the expectations of the kgprotocol
* Support both compressed and uncompressed public keys (0x03 and 0x04 leading bytes)
* Store private keys in encrypted form only

### Non-technical:

* Safeguard the privacy of the communicating parties (no user/device data-gathering, no traffic or location analysis)
* Remain free of dependency on cellular/wifi service
* Remain free of dependency on proprietary or closed services
* Retain responsibility for key generation, naming and public key sharing with the keymaster user
* Any keymaster/gatekeeper implementation that implements kgprotocol may claim ADILOS compliance