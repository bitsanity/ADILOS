**UPDATE**: add [adiweb](https://github.com/bitsanity/adiweb) as another example of a kgagent.

UPDATE: added [simpleth](https://github.com/bitsanity/simpleth) as a newer version of keymaster.

UPDATE: moved adilos.js to an npm-compatible module. See [adilosjs](https://github.com/bitsanity/adilosjs)

UPDATE: added adilos.js to help javascript-based clients use ADILOS protocol

UPDATE: ADILOS is an effective Self-Sovereign Identity System

UPDATE: added video illustration of identification approach. See [Cryptographic SSN](https://youtu.be/vU9M11hJgXo)

UPDATE: added ECIES encryption between kgagent and kgserver  

ADILOS enables computer device users to gain access to resources. ADILOS uses a well-known cryptographic approach to perform a public-key exchange over an open channel.

ADILOS assumes an authorization process in which a user's public key is recorded in an ACL (access-control list) using an out-of-band communication. For a user this simply means displaying a public key so it can be entered in the ACL.

The reference implementation demonstrates data exchange using QR (quick read) optical codes scanned using high-definition cameras. This enables one to use a personal device such as a "smart" cellphone to obtain access.

ADILOS contains these subsystems:

* keymaster
* gatekeeper
* keymaster-gatekeeper communications interface ("kgprotocol")
* kgagent
* kgserver
* simpleth
* adiweb

## 1. keymaster

A reference implementation of a keymaster, for Android.

See [bitsanity / keymaster](https://github.com/bitsanity/keymaster)

## 2. gatekeeper

A reference implementation of a gatekeeper, for Raspbian Linux on Raspberry Pi 3.

See [bitsanity / gatekeeper](https://github.com/bitsanity/gatekeeper)

## 3. kgprotocol

Described in detail later in this document. Best to start with the videos on the [wiki](https://github.com/bitsanity/ADILOS/wiki) page.

## 4. kgagent

A gateway that acts between a keymaster and a kgserver. The agent interacts with a keymaster to sign a server's challenge. Then the agent communicates with the kgserver to retrieve server resources.

See [bitsanity / kgagent](https://github.com/bitsanity/kgagent)

## 5. kgserver

A network/web service that presents an authentication challenge to a connecting kgagent. Once the response is received the server knows the keymaster's public key and exchanges JSON-RPC messages with the agent on that basis.

See [bitsanity / kgserver](https://github.com/bitsanity/kgserver)

## 6. simpleth

This is another Android app like keymaster and replaces keymaster as our reference client app. Simpleth is 100% script (no native code!)
and adds the ability to sign a 32-byte value

See [bitsanity / simpleth](https://github.com/bitsanity/simpleth)

## 7. adiweb

This is a demonstration XML-RPC service and kgagent that provides authenticated and encrypted communications without relying on HTTPS or
any centralized certification authority.

See [bitsanity / adiweb](https://github.com/bitsanity/adiweb)

## A. Expectations for any **keymaster**

* Must work equally well with any kgprotocol-compliant gatekeeper
* Secure the app and its data from unauthorized use (e.g. require PIN/passphrase on start)

## B. Expectations for any **gatekeeper**

* Must work equally well with any kgprotocol-compliant keymaster
* May maintain an ACL (access control list) of authorized public keys
* May maintain relationships between a public key and other user-provided information
* Must generate a new, random key for each Challenge
* Must generate a new Challenge on start and after receipt of any cryptographically-valid Response

## C. kgprotocol Illustration

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
| 5    | approach gatekeeper (or use kgagent) | |
| 6    | select public key to use | |
| 7    | scan Challenge | |
| 8    | verify the challenge (check digital signature) | |
| 9    | if challenge valid, generate Response using selected key | |
| 10    | encode and display Response | |
| 10   | share Response with gatekeeper (or kgagent and kgserver) | |
| 11   | | scan and decode Response |
| 12   | | verify response (check digital signature) | |
| 13   | | parse public key from Response |
| 14   | | look up public key in ACL |
| 15   | | depending on result of lookup, allow or deny access |
| 16   | | return to step 1 |


