# Bitcoin Basic HTLC Atomic Swap

- RFC-Number: 005
- Status: Draft
- Discussion-issue: [#42](https://github.com/comit-network/RFCs/issues/42)
- Created on: 01 Mar. 2019

# Table of Contents

<!-- markdown-toc start -->
- [Description](#description)
- [The Bitcoin Identity](#the-bitcoin-identity)
- [Hash Time Lock Contract](#hash-time-lock-contract)
    - [Hash Functions](#hash-functions)
    - [Parameters](#parameters)
    - [Contract](#contract)
- [Execution Phase](#execution-phase)
    - [Deployment](#deployment)
    - [Redeem](#redeem)
    - [Refund](#refund)
- [Registry extension](#registry-extension)
- [Examples/Test vectors](#examplestest-vectors)
    - [RFC003 SWAP REQUEST](#rfc003-swap-request)
    - [RFC003 SWAP RESPONSE](#rfc003-swap-response)
    - [HTLC](#htlc)

<!-- markdown-toc end -->

## Description

This RFC defines how to execute a [RFC003](./RFC-003-SWAP-Basic.md) SWAP where one of the ledgers is Bitcoin and the associated asset is the native Bitcoin asset.

For definitions of the Bitcoin ledger and asset see [RFC004](./RFC-004-Bitcoin.md).

To fulfil the requirements of [RFC003](./RFC-003-SWAP-Basic.md) this RFC defines:

- The [identity](./RFC-003-SWAP-Basic.md#identity) to be used when negotiating a SWAP on the Bitcoin ledger.
- How to construct a Hash Time Lock Contract (HTLC) to lock the Bitcoin asset on the Bitcoin blockchain.
- How to deploy, redeem and refund the HTLC during the execution phase of [RFC003](./RFC-003-SWAP-Basic.md).

## The Bitcoin Identity

[RFC003](./RFC-003-SWAP-Basic.md) requires ledgers have an *identity* type specified to negotiate a SWAP.

The Identity to be used on Bitcoin is the 20-byte *pubkeyhash* which is the result of applying SHA-256 and then RIPEMD-160 to a user's compressed SECP256k1 public key.
The compressed public key is used because it needs to be compatible with segwit transactions (see [BIP143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Restrictions_on_public_key_type)).

While it may seem more intuitive to use a Bitcoin *address* as the identity, the pubkeyhash better fits the definition of identity given in [RFC003](./RFC-003-SWAP-Basic.md).
Using a Bitcoin address as the identity would require implementations to do a number of cumbersome validation steps such as verifying that it is a p2pkh or p2wpkh address, extracting the pubkeyhash and validating the network.

In the JSON encoding, a *pubkeyhash* MUST be encoded as a 20-byte hex string.

This RFC extends the [registry](./registry.md) with the following entry in the identity table:

| Ledger   | Identity Name | JSON Encoding            | Description                                                                                  |
|:----     |:-------       |:-------------            | -------------------------------------------------------------------------------------------- |
| Bitcoin  | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA-256 and then RIPEMD-160 to a user's SECP256k1 compressed public key |

## Hash Time Lock Contract

### Hash Functions

This RFC specifies SHA-256 as the only value the `hash_function` parameter to `comit-rfc-003` may take if Bitcoin is used as a ledger.
This may be expanded in subsequent RFCs.

### Parameters

The parameters for the Bitcoin HTLC follow [RFC003](./RFC-003-SWAP-Basic.md#hash-time-lock-contract-htlc) and are described concretely in the following table:

| Variable        | Description                                                                 |
|:----------------|:----------------------------------------------------------------------------|
| asset           | The quantity of satoshi                                                     |
| secret_hash     | The hash of the `secret` (32 bytes for SHA-256)                             |
| redeem_identity | The `pubkeyhash` of the redeeming party                                     |
| refund_identity | The `pubkeyhash` of the refunding party                                     |
| expiry          | The absolute UNIX timestamp in seconds after which the HTLC can be refunded |

### Contract

The HTLC is constructed by locking an output with the following script:

```
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

As required by [RFC003](./RFC-003-SWAP-Basic.md), this HTLC uses absolute time locks to check whether `expiry` has been reached.
Specifically, `OP_CHECKLOCKTIMEVERIFY` (see [BIP65](https://github.com/bitcoin/bips/blob/master/bip-0065.mediawiki)) is used to compare the time in the block's header to the `expiry` in the contract.

Implementations MUST consider an `expiry` value below `500000000` for a Bitcoin HTLC to be invalid due to the [`lock_time`](https://en.bitcoin.it/wiki/Protocol_documentation#tx) transaction field interpreting values below `500000000` as block heights.

To compute the exact Bitcoin script bytes of the contract, implementations should use the following offset table:

| Data              | Position of first byte | Position of last byte | Length (bytes) | Description                                                                |
|:------------------|:-----------------------|:----------------------|:---------------|:---------------------------------------------------------------------------|
| `6382012088a820`  | 0                      | 6                     | 7              | `OP_IF OP_SIZE 32 OP_EQUALVERIFY OP_SHA256` + `PUSH32` for the secret_hash |
| `secret_hash`     | 7                      | 39                    | 32             | See [Parameters](#parameters)                                              |
| `8876a9`          | 40                     | 42                    | 3              | `OP_EQUALVERIFY OP_DUP OP_HASH160`                                         |
| `redeem_identity` | 43                     | 64                    | 20             | See [Parameters](#parameters)                                              |
| `67`              | 65                     | 65                    | 1              | `OP_ELSE`                                                                  |
| `expiry`          | 66                     | 69                    | 4              | See [Parameters](#parameters)                                              |
| `b17576a9`        | 70                     | 73                    | 4              | `OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160`                         |
| `refund_identity` | 74                     | 93                    | 20             | See [Parameters](#parameters)                                              |
| `6888ac`          | 94                     | 96                    | 3              | `OP_ENDIF OP_EQUALVERIFY OP_CHECKSIG`                                      |


## Execution Phase

The following section describes how both parties should interact with the Bitcoin blockchain during the [RFC003 execution phase](./RFC-003-SWAP-Basic.md#execution-phase).

### Deployment

At the start of the deployment stage, both parties compile the contract as described in the previous section.
We will call this value `contract_script`.

To deploy the Bitcoin HTLC, the *funder* must confirm a transaction on the relevant Bitcoin blockchain.
One of the transaction's outputs must have the following properties:

- Its `value` MUST be equal to the `quantity` parameter in the Bitcoin asset header.
- It MUST have a Pay-To-Witness-Script-Hash (P2WSH) `scriptPubKey` derived from `contract_script` (See [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#specification) for how to construct the `scriptPubkey` from the `contract_script`).

To be notified of the deployment event, both parties MAY watch the blockchain for a transaction with an output matching the required `scriptPubkey` and having the required value.

### Redeem

Before redeeming, *the redeemer* SHOULD wait until the deployment transaction is included in the Bitcoin blockchain with enough confirmations such that they consider it permanent.

To redeem the HTLC, the redeemer MUST submit a transaction to the blockchain which spends the P2WSH output.
The redeemer can use following witness data to spend the output if they know the `secret`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| redeem_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `redeem_pubkey`    |
| redeem_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `redeem_identity` |
| secret           | The pre-image of the `secret_hash` under the `hash_function`                                            |
| `01`             | A single byte used to activate the redeem path in the `OP_IF`                                           |
| contract_script  | The compiled contract (as generally required when redeeming from a P2WSH output)                        |

For how to use this witness data to construct the redeem transaction see [BIP141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki#transaction-id).

To be notified of the redeem event, both parties MAY watch the blockchain for transactions that spend from the output and check that the witness data is in the above form.
If Bitcoin is the `beta_ledger` (see [RFC003](./RFC-003-SWAP-Basic.md)), then the funder MUST watch for such a transaction and  extract the `secret` from its witness data and continue the protocol.

### Refund

To refund the HTLC, the funder MUST submit a transaction to the blockchain which spends the P2WSH output.
The funder can use the following witness data to spend the output after the `expiry`:

| Data             | Description                                                                                             |
|:-----------------|:--------------------------------------------------------------------------------------------------------|
| refund_signature | A valid SECP256k1 ECDSA DER encoded signature on the transaction with respect to the `refund_pubkey`    |
| refund_pubkey    | The 33 byte SECP256k1 compressed public key that was hashed to produce the pubkeyhash `refund_identity` |
| `00`             | A single byte used to activate the refund path in the `OP_IF`                                           |
| contract_script  | The compiled contract (as generally required when redeeming from a P2WSH output)                        |

To be notified of the refund event, both parties MAY watch the blockchain for transactions that spend from the output and check that the witness data is in the above form.

## Registry extension

This RFC extends the [registry](./registry.md#identities) with an identity definition for the Bitcoin ledger:

| Ledger  | Identity Name | JSON Encoding            | Description                                                                           |
|:--------|:--------------|:-------------------------|---------------------------------------------------------------------------------------|
| Bitcoin | `pubkeyhash`  | `hex-encoded-bytes (20)` | The result of applying SHA-256 and then RIPEMD-160 to a SECP256k1 compressed public key |

# Examples/Test vectors

## RFC003 SWAP REQUEST

The following shows an [RFC003](RFC-003-SWAP-Basic.md) SWAP REQUEST where the `alpha_ledger` is Bitcoin, the `alpha_asset` is 1 Bitcoin (with `...` being used where the value is only relevant for the `beta_ledger`).

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
    "protocol": {
        "value" : "comit-rfc-003",
        "parameters" : { "hash_function" : "SHA-256" }
    }
  },
  "body": {
    "aplha_ledger_refund_identity": "1925a274ac004373bb5429553bdb55c40e57b124",
    "alpha_expiry": 1552263040,
    "secret_hash" : "1f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e",
    "beta_ledger_redeem_identity" : "...",
    "beta_expiry" : ...
  },
}
```

Note, the secret for the `secret_hash` is `51a488e06e9c69c555b8ad5e2c4629bb3135b96accd1f23451af75e06d3aee9c`.

## RFC003 SWAP RESPONSE
A valid `RESPONSE` to the above `REQUEST` could look like:

``` json
{
  "status" : "OK00",
  "body": {
     "alpha_ledger_redeem_identity": "c021f17be99c6adfbcba5d38ee0d292c0399d2f5",
     "beta_ledger_refund_identity": "..."
  }
}
```


## HTLC

The above `REQUEST` and `RESPONSE` results in the following parameters to the HTLC:

| Parameter       | value                                                              |
|:----------------|--------------------------------------------------------------------|
| redeem_identity | `c021f17be99c6adfbcba5d38ee0d292c0399d2f5`                         |
| redund_identity | `1925a274ac004373bb5429553bdb55c40e57b124`                         |
| secret_hash     | `1f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e` |
| expiry          | 1552263040                                                         |


Which compiles into the following Bitcoin script bytes:

```
6382012088a8201f69c8745f712da03fdd43486ef705fc24f3e34d54cf44d967cf5cd4204c835e8876a914c021f17be99c6adfbcba5d38ee0d292c0399d2f5670480a7855cb17576a9141925a274ac004373bb5429553bdb55c40e57b1246888ac
```

Which results in the following P2WSH address by network:

| Network   | Address                                                            |
|:----------|--------------------------------------------------------------------|
| `regtest` | `bcrt1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90ws9hj4q3` |
| `testnet` | `tb1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90wsgwcn4t`   |
| `mainnet` | `bc1q4vft3swvhm5zvytlsx0puwsge7pnsj4zmvwp9gcyvwhnuthn90wslxwu0y`   |
