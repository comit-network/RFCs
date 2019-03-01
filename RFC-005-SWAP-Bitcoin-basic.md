# Bitcoin Basic HTLC Atomic Swap

- RFC-Number: 005
- Status: Draft
- Discussion-issue: -
- Created on: 01 Mar. 2019

# Table of contents

<!-- toc -->

- [Description](#description)
- [Motivation](#motivation)
- [Content](#content)
  - [Identities](#identities)
  - [Scripts](#scripts)
    - [Variables](#variables)
    - [HTLC](#htlc)
      - [Human friendly - Opcode representation](#human-friendly---opcode-representation)
      - [Implementation friendly - Hex representation](#implementation-friendly---hex-representation)
    - [Redeem](#redeem)
    - [Refund](#refund)
    - [Timeouts](#timeouts)
- [Registry extension](#registry-extension)

<!-- tocstop -->

## Description

This RFC introduces the Hashed TimeLock Contract and identity to allow the execution of a [Basic HTLC Atomic Swap](./RFC-003-SWAP-basic.md) involving [Bitcoin Core](https://github.com/bitcoin/bitcoin/).

<!-- TODO: Mention P2SH as open question in RFC tracking issue -->
TODO: based on both TODO above, decide to include version and/or BIPs to be enabled.
This includes the exact Bitcoin scripts to be used.

## Motivation

This RFC aims to define the Bitcoin Script and other needed element to support Bitcoin Core in [Basic HTLC Atomic Swap](./RFC-003-SWAP-basic.md).

## Content

### Identities

While it may seem obvious at first to use an address as an identity, such choice would come with a number of unnecessary manipulation and checks to validate the address. Such as verifying that it is a p2pkh or p2wpkh address, extraction of the public key hash, validation of the network.

For simplicity purposes, the identity on bitcoin is defined as the public key hash. It is the same public key hash (SHA256+RIPEMD160) that is used to construct addresses, ensuring similar security level to sharing an address.

Note that only compressed public keys are accepted as it needs to be usable in P2WSH[1].

| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 compressed public key |

### Scripts

#### Variables

The information in the table is needed to build the HTLC.

the `[20]` denotation is used to express a push of the value between bracket onto the stack. Note that, as per usual Bitcoin script foramt, all values and placeholder presents MUST be encoded in hexadecimal without a leading `0x`. E.g. `[20]` would be replaced with `01 20` to push the decimal number `32` on the stack.

| Variable | Description |
| `secret` | |
| `secret_hash` | The SHA-256 hash of the `secret` |
| `recipient_identity` | The identity of the recipient, as defined in [Identities](#identities), in case of redeem |
| `sender_identity` | The identity of the sender, as defined in [Identities](#identities), in case of refund |
| `timestamp` | The absolute UNIX timestamp in seconds after which the HTLC can be refunded |

#### HTLC

The Bitcoin script we purpose is a P2WSH (Pay-to-Witness-Script-Hash), meaning a Bitcoin Basic HTLC Atomic Swap can only be executed on a Bitcoin node with Segregated Witness activated[2].

##### Human friendly - Opcode representation

The HTLC is as below in Opcode format. 

Note that an implementation SHOULD not relay on this format but instead use the .... 

```asm
OP_IF
OP_SIZE
[20]
OP_EQUALVERIFY 
OP_SHA256
[secret_hash]
OP_EQUALVERIFY
OP_DUP
OP_HASH160
[recipient_identity]
OP_ELSE
[timestamp]
OP_CHECKLOCKTIMEVERIFY
OP_DROP
OP_DUP
OP_HASH160
[sender_identity]
OP_ENDIF
OP_EQUALVERIFY
OP_CHECKSIG
```

##### Implementation friendly - Hex representation

The hexadecimal representation of the HTLC SHOULD be used to implement [RFC-003](./RFC-003-SWAP-basic.md) for Bitcoin.

Replacement of placeholders SHOULD NOT be used, instead, building by concatenation or byte position is recommended for security purposes. See [TODO](#security-concern) for more details.

Use the following steps to build the script:
<!-- TODO: Verify this is correct! -->
| Data                 | Position of first Byte | Position of last Byte| Length | Description                                                                  |
|:---                  |:---                    |:---                  |:---    |:---                                                                          |
| `6382012088a820`     | 0                      | 6                    | 7      | `OP_IF OP_SIZE [20] OP_EQUALVERIFY OP_SHA256` + `PUSH32` for the secret_hash |
| `secret_hash`        | 7                      | 39                   | 32     | See [Variables](#variables)                                                  |
| `8876a9`             | 40                     | 42                   | 3      | `OP_EQUALVERIFY OP_DUP OP_HASH160`                                           |
| `recipient_identity` | 43                     | 74                   | 32     | See [Variables](#variables)                                                  |
| `67`                 | 75                     | 75                   | 1      | `OP_ELSE`                                                                    |
| `timestamp`          | 76                     | 79                   | 4      | See [Variables](#variables)                                                  |
| `b17576a9`           | 80                     | 83                   | 4      | `OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160`                           |
| `sender_identity`    | 84                     | 115                  | 32     | See [Variables](#variables)                                                  |
| `6888ac`             | 116                    | 118                  | 3      | `OP_ENDIF OP_EQUALVERIFY OP_CHECKSIG`                                        |


#### Redeem
Describe redeem script

#### Refund
Describe refund script and any caveat/things to watch out

#### Timeouts

TODO: Do we want to discuss timeouts?


## Registry extension

A list of changes to the registry described in this RFC. For example, new protocols, hash functions etc.

---

- [1] Compressed keys in P2WSH: See [BIP143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Restrictions_on_public_key_type)
- [2] Segregated Witness: [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki)
