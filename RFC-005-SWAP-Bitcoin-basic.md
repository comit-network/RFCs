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
}- [Registry extension](#registry-extension)

<!-- tocstop -->

## Description

This RFC defines the Bitcoin ledger and the Bitcoin asset for use in an execution [RFC002](./RFC-002-SWAP.md) SWAP with `comit-rfc-003` ([RFC003](./RFC-003-SWAP-basic.md)) as the SWAP protocol.

For definitions of the Bitcoin ledger and asset see [RFC004](./RFC-004-SWAP-Bitcoin).

To fulfil the requirements of RFC003 this RFC defines:

- The *identity* to be used when negotiating a SWAP on the Bitcoin ledger.
- How to construct a Hash Time Lock Contract (HTLC) to lock the Bitcoin asset on the Bitcoin blockchain.
- How to deploy, redeem and refund the HTLC during the execution phase of RFC003


## Motivation

The motivation for creating this RFC is to allow implementations to negotiate and execute [RFC003 Basic HTLC Atomic Swaps](./RFC-003-SWAP-basic.md) using the COMIT protocol. The identity definition introduced in this RFC should also be useful for future protocols.

## Bitcoin Identity

The Identity to be used on Bitcoin is the 20-byte *pubkeyhash* which is the result of applying SHA-256 and then RIPEMD-160 to a user's compressed SECP256k1 public key.
The compressed public key is used because it needs to be compatiable with segwit transactions (see [BIP143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Restrictions_on_public_key_type)).

While it may seem more intuitive to use a Bitcoin *address* as the identity, the pubkeyhash better fits the definition of identity given in RFC003. Using a Bitcoin address as the identity would require implementations to do a number of cumbersome validation steps such as such as verifying that it is a p2pkh or p2wpkh address, extraction of the pubkeyhash, validation of the network.

When using the pubkeyhash as one of the identities in an RFC003 message body it MUST be encoded as 20 hex encoded string.
This RFC extends the [registry](./registry.md) with the following entry in the identity table:

| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 compressed public key |

## Hash Time Lock Contract

### Hash Functions


The `hash_function` header must be set to one of the following values if Bitcoin is used as a ledger:

- SHA-256

### Parameters

The parameters for the Bitcoin HTLC follow [RFC003](./RFC-003-SWAP-basic.md) and are described concretely in the following table:

| Variable        | Description                                                                                                               |
|:----------------|:--------------------------------------------------------------------------------------------------------------------------|
| asset           | The quantity of satoshi                                                                                                   |
| secret          | The secret (32 bytes for SHA-256)                                                                                         |
| secret_hash     | The SHA-256 hash of the `secret` (32 bytes for SHA-256)                                                                   |
| redeem_identity | The `pubkeyhash` of the redeeming parry                                                                                   |
| refund_identity | The `pubkeyhash` of the refunding party                                                                                   |
| expiry          | The absolute UNIX timestamp in seconds after which the HTLC can be refunded |

### Contract

The HTLC is constructed by locking an output with the following script:

```asm
OP_IF
    OP_SIZE 32 OP_EQUALVERIFY
    OP_SHA256 <secret_hash> OP_EQUALVERIFY
    OP_DUP OP_HASH160 <redeem_identity>
OP_ELSE
    <timestamp> OP_CHECKLOCKTIMEVERIFY OP_DROP
    OP_DUP OP_HASH160 <refund_identity>
OP_ENDIF
OP_EQUALVERIFY
OP_CHECKSIG
```

As required by RFC003, this HTLC uses absolute time locks to check whether `expiry` has been reached.
Specifically, `OP_CHECKLOCKTIMEVERIFY` (see [BIP65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki)) is used to compare the time in the block's header to the `expiry` in the contract.

To compute the exact bytes of the contract implementaions should use the following offset table:

| Data              | Position of first byte | Position of last byte | Length | Description                                                                |
|:------------------|:-----------------------|:----------------------|:-------|:---------------------------------------------------------------------------|
| `6382012088a820`  | 0                      | 6                     | 7      | `OP_IF OP_SIZE 32 OP_EQUALVERIFY OP_SHA256` + `PUSH32` for the secret_hash |
| `secret_hash`     | 7                      | 39                    | 32     | See [Parameters](#parameters)                                              |
| `8876a9`          | 40                     | 42                    | 3      | `OP_EQUALVERIFY OP_DUP OP_HASH160`                                         |
| `redeem_identity` | 43                     | 74                    | 32     | See [Parameters](#parameters)                                              |
| `67`              | 75                     | 75                    | 1      | `OP_ELSE`                                                                  |
| `timestamp`       | 76                     | 79                    | 4      | See [Parameters](#parameters)                                              |
| `b17576a9`        | 80                     | 83                    | 4      | `OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160`                         |
| `refund_identity` | 84                     | 115                   | 32     | See [Parameters](#parameters)                                              |
| `6888ac`          | 116                    | 118                   | 3      | `OP_ENDIF OP_EQUALVERIFY OP_CHECKSIG`                                      |


## Execution Phase

### Deployment

The party who is required to deploy the Bitcoin HTLC (the refunder) compiles the contract as described in the previous section.
Then, they MUST send a transaction to the Bitcoin Blockchain with an output with value exactly equal to the quantity of satoshi specified by `asset` and a Pay-To-Witness-Script-Hash (P2WSH) `scriptPubKey` derived from the contract. See [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#specification)
for how to construct the `scriptPubkey`.

The redeeming party should likewise compile the contract script and wait until the above transaction appears in the Bitcoin blockchain with enough confirmations such that they consider it permanent. They should do this by watching the Blockchain for a transaction with an output matching the `scriptPubkey` and having the required value.

#### Redeem

To redeem from the HTLC, the redeemer should submit a transaction to the blockchain which spends the P2WSH output.
The redeemer can use following witness data to spend the output if they know the `secret`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| redeem_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `redeem_pubkey`    |
| redeem_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `redeem_identity` |
| secret           | The secret                                                                                              |
| `01`             | Used to activate the redeem path in the `OP_IF`                                                         |
| contract         | The compiled contract (as generally required when redeeming from a P2WSH output)                        |

For how to use this to construct the redeem transaction see [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#transaction-id).

#### Refund

To refund the HTLC, the refunder should submit a transaction to the blockchain which spends the P2WSH output.
The refunder can use the following witness data to spend the output after the `expiry`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| refund_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `refund_pubkey`    |
| redeem_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `redeem_identity` |
| `00`              | Used to activate the refund path in the `OP_IF`                                                         |
| contract         | The compiled contract (as generally required when redeeming from a P2WSH output)                        |


## Registry extension

This RFC extends the [registry](./registry.md) in the following ways:

- **pubkeyhash**: The identities table now includes:
| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 compressed public key |


# Examples

## Example RFC003 message body

TODO


## HTLC Test Vectors


TODO put some example test vectors for the HTLC script
