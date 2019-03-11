# Bitcoin Basic HTLC Atomic Swap

- RFC-Number: 005
- Status: Draft
- Discussion-issue: -
- Created on: 01 Mar. 2019

# Table of Contents
<!-- markdown-toc start -->

- [Bitcoin Basic HTLC Atomic Swap](#bitcoin-basic-htlc-atomic-swap)
- [Table of contents](#table-of-contents)
    - [Description](#description)
    - [Motivation](#motivation)
    - [Bitcoin Identity](#bitcoin-identity)
    - [Hash Time Lock Contract](#hash-time-lock-contract)
        - [Hash Functions](#hash-functions)
        - [Parameters](#parameters)
        - [Contract](#contract)
    - [Execution Phase](#execution-phase)
        - [Deployment](#deployment)
            - [Redeem](#redeem)
            - [Refund](#refund)
    - [Registry extension](#registry-extension)
- [Examples](#examples)
    - [Example RFC003 message body](#example-rfc003-message-body)
    - [HTLC Test Vectors](#htlc-test-vectors)

<!-- markdown-toc end -->

## Description

This RFC defines how to execute a ([RFC003](./RFC-003-SWAP-basic.md)) SWAP where one of the ledgers is Bitcoin and the associated asset is the native Bitcoin asset.

For definitions of the Bitcoin ledger and asset see [RFC004](./RFC-004-SWAP-Bitcoin).

To fulfil the requirements of RFC003 this RFC defines:

- The [identity](./RFC-003-SWAP-basic.md#identity) to be used when negotiating a SWAP on the Bitcoin ledger.
- How to construct a Hash Time Lock Contract (HTLC) to lock the Bitcoin asset on the Bitcoin blockchain.
- How to deploy, redeem and refund the HTLC during the execution phase of RFC003.

## The Bitcoin Identity

The Identity to be used on Bitcoin is the 20-byte *pubkeyhash* which is the result of applying SHA-256 and then RIPEMD-160 to a user's compressed SECP256k1 public key.
The compressed public key is used because it needs to be compatible with segwit transactions (see [BIP143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Restrictions_on_public_key_type)).

While it may seem more intuitive to use a Bitcoin *address* as the identity, the pubkeyhash better fits the definition of identity given in RFC003.
Using a Bitcoin address as the identity would require implementations to do a number of cumbersome validation steps such as verifying that it is a p2pkh or p2wpkh address, extracting the pubkeyhash and validating the network.

When using the pubkeyhash as one of the identities in an RFC003 message body it MUST be encoded as 20 hex encoded string.
This RFC extends the [registry](./registry.md) with the following entry in the identity table:

| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a user's SECP256k1 compressed public key |

## Hash Time Lock Contract

### Hash Functions

SHA-256 is the only value the `hash_function` header may take if Bitcoin is used as a ledger.

### Parameters

The parameters for the Bitcoin HTLC follow [RFC003](./RFC-003-SWAP-basic.md) and are described concretely in the following table:

| Variable        | Description                                                                 |
|:----------------|:----------------------------------------------------------------------------|
| asset           | The quantity of satoshi                                                     |
| secret_hash     | The hash of the `secret` (32 bytes for SHA-256)                             |
| redeem_identity | The `pubkeyhash` requ the redeeming party                                   |
| refund_identity | The `pubkeyhash` of the refunding party                                     |
| expiry          | The absolute UNIX timestamp in seconds after which the HTLC can be refunded |

### Contract

The HTLC is constructed by locking an output with the following script:

```asm
OP_IF
    OP_SIZE 32 OP_EQUALVERIFY
    OP_SHA256 <secret_hash> OP_EQUALVERIFY
    OP_DUP OP_HASH160 <redeem_identity>
OP_ELSE
    <expiry> OP_CHECKLOCKTIMEVERIFY OP_DROP
    OP_DUP OP_HASH160 <refund_identity>
OP_ENDIF
OP_EQUALVERIFY
OP_CHECKSIG
```

As required by RFC003, this HTLC uses absolute time locks to check whether `expiry` has been reached.
Specifically, `OP_CHECKLOCKTIMEVERIFY` (see [BIP65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki)) is used to compare the time in the block's header to the `expiry` in the contract.
Due to [`lock_time`](https://en.bitcoin.it/wiki/Protocol_documentation#tx) transaction field interpreting values below `500000000` as block heights, implementations MUST consider such an `expiry` invalid.

To compute the exact bytes of the contract implementations should use the following offset table:

| Data              | Position of first byte | Position of last byte | Length | Description                                                                |
|:------------------|:-----------------------|:----------------------|:-------|:---------------------------------------------------------------------------|
| `6382012088a820`  | 0                      | 6                     | 7      | `OP_IF OP_SIZE 32 OP_EQUALVERIFY OP_SHA256` + `PUSH32` for the secret_hash |
| `secret_hash`     | 7                      | 39                    | 32     | See [Parameters](#parameters)                                              |
| `8876a9`          | 40                     | 42                    | 3      | `OP_EQUALVERIFY OP_DUP OP_HASH160`                                         |
| `redeem_identity` | 43                     | 64                    | 20     | See [Parameters](#parameters)                                              |
| `67`              | 65                     | 65                    | 1      | `OP_ELSE`                                                                  |
| `expiry`          | 66                     | 69                    | 4      | See [Parameters](#parameters)                                              |
| `b17576a9`        | 70                     | 73                    | 4      | `OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160`                         |
| `refund_identity` | 74                     | 93                    | 20     | See [Parameters](#parameters)                                              |
| `6888ac`          | 94                     | 96                    | 3      | `OP_ENDIF OP_EQUALVERIFY OP_CHECKSIG`                                      |


## Execution Phase

The following section describes how both parties should interact with the Bitcoin blockchain during the [RFC003 execution phase](./RFC-003-SWAP-basic#execution-phase).

### Deployment

The party who is required to deploy the Bitcoin HTLC (the funder) compiles the contract into bytes as described in the previous section.
To deploy it, they send a transaction to the Bitcoin blockchain with an output whose value exactly equal to the `quantity` parameter in the relevant asset header and a Pay-To-Witness-Script-Hash (P2WSH) `scriptPubKey` derived from the contract.
See [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#specification) for how to construct the `scriptPubkey` from the contract bytes.

The redeeming party (the redeemer) should likewise compile the contract script and wait until the above transaction appears in the Bitcoin blockchain with enough confirmations such that they consider it permanent.
They MAY do this by watching the blockchain for a transaction with an output matching the `scriptPubkey` and having the required value.

#### Redeem

To redeem from the HTLC, the redeemer should submit a transaction to the blockchain which spends the P2WSH output.
The redeemer can use following witness data to spend the output if they know the `secret`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| redeem_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `redeem_pubkey`    |
| redeem_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `redeem_identity` |
| secret           | The pre-image to the `secret_hash` used to generate the HTLC                                            |
| `01`             | A single byte used to activate the redeem path in the `OP_IF`                                           |
| contract         | The compiled contract (as generally required when redeeming from a P2WSH output)                        |

For how to use this to construct the redeem transaction see [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#transaction-id).

To be notified of the redeem event both parties may watch the blockchain for transactions that spend from the output and check that the witness data is in the above form.
If Bitcoin is the `beta_ledger`, then the funder (Bob) MUST watch for such a transaction and then extract the `secret` from the witness data and continue the protocol.

#### Refund

To refund the HTLC, the funder should submit a transaction to the blockchain which spends the P2WSH output.
The funder can use the following witness data to spend the output after the `expiry`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| refund_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `refund_pubkey`    |
| refund_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `refund_identity` |
| `00`             | A single byte used to activate the refund path in the `OP_IF`                                           |
| contract         | The compiled contract (as generally required when redeeming from a P2WSH output)                        |

To be notified of the refund event both parties may watch the blockchain for transactions that spend from the output and check that the witness data is in the above form.

## Registry extension

This RFC extends the [registry](./registry.md) with an identity definition for the Bitcoin ledger:

| Ledger  | Identity Name | JSON Encoding            | Description                                                                           |
|:--------|:--------------|:-------------------------|---------------------------------------------------------------------------------------|
| Bitcoin | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA256 and then RIPEMD160 to a SECP256k1 compressed public key |

# Examples

## Example RFC003 message body

This following shows what a SWAP message looks like where the `alpha_ledger` is Bitcoin, the `alpha_asset` is 1 Bitcoin (with `...` being used where the value is only relevant for the `beta_ledger`).

``` json
{
  "type": "SWAP",
  "headers": {
    "alpha_ledger": {
      "value": "bitcoin",
      "parameters": { "network": "mainnet" }
    },
    "beta_ledger": {...},
    "alpha_asset": {
      "value": "bitcoin",
      "parameters": { "quantity": "100000000" }
    },
    "beta_asset": {...},
    "protocol": "comit-rfc-003",
  },
  "body": {
    "aplha_ledger_refund_identity": "1925a274ac004373bb5429553bdb55c40e57b124",
    "alpha_expiry": 123456789,
    "hash_function": "SHA-256",
    "secret_hash" : "51a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c",
    "beta_ledger_redeem_identity" : "...",
    "beta_expiry" : ...
  },
}
```

And a valid `RESPONSE` could look like:

``` json
{
  "body": {
     "alpha_ledger_redeem_identity": "c021f17be99c6adfbcba5d38ee0d292c0399d2f5",
     "beta_ledger_refund_identity": "...",
  }
}
```

Which would give us the following parameters for the HTLC:

| Parameter       | value                                                              |
|:----------------|--------------------------------------------------------------------|
| redeem_identity | `c021f17be99c6adfbcba5d38ee0d292c0399d2f5`                         |
| redund_identity | `1925a274ac004373bb5429553bdb55c40e57b124`                         |
| secret_hash     | `51a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c` |
| expiry          | 123456789                                                          |

Both parties should be able to compile the HTLC into this byte sequence:

```
6382012088a82051a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c8876a914c021f17be99c6adfbcba5d38ee0d292c0399d2f5670415cd5b07b17576a9141925a274ac004373bb5429553bdb55c40e57b1246888ac
```
